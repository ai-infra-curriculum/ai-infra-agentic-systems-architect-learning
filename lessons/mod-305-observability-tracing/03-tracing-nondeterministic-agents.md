# Chapter 3 — Tracing Non-Deterministic, Long-Running Agents

A request-response service is easy to trace because each request is self-contained, short, and reproducible. Agents are none of those things. A run retries, branches, and loops; a "session" can span hours and many runs; and the same input never reproduces, so you can't debug by replay. The architecture that handles a stateless web request will mislead you here. This chapter is about the three identity and lifecycle problems that are specific to non-deterministic, long-running agents — and the design decisions that solve them.

## Problem 1 — Trace identity across retries and loops

An agent loop calls the model, the model asks for a tool, you run the tool, you call the model again — N times until it stops. If a transient error triggers a retry, you now have *two attempts at the same logical step*. Naïve instrumentation either collapses them into one span (you lose the retry) or scatters them across two traces (you lose the relationship). Neither is debuggable.

The design rule: **one logical agent run is exactly one trace; every attempt within it is a span under that trace.** The trace is the run; retries are siblings, not new traces.

```text
trace_id = T  (one logical run)
invoke_agent  worker[research]
├── chat  attempt=1   status=ERROR  (rate-limited)
├── chat  attempt=2   status=ERROR  (rate-limited)
└── chat  attempt=3   status=OK      ← the retry chain is visible, in order
```

Carry an explicit `retry.attempt` attribute and an idempotency key so you can tell "the agent legitimately called the search tool three times for three queries" from "the agent retried one failing search three times." Without the attribute those look identical in the span tree and mean opposite things.

The loop itself needs a guard rail that is also an observability signal. Emit the **iteration count** as a span attribute and alert when it approaches the cap:

```python
MAX_ITERATIONS = 12

with tracer.start_as_current_span(f"invoke_agent {role}") as span:
    for i in range(MAX_ITERATIONS):
        span.set_attribute("agent.loop.iteration", i)
        step = await agent_step(state)
        if step.is_final:
            span.set_attribute("agent.loop.terminated", "final")
            break
    else:
        span.set_attribute("agent.loop.terminated", "max_iterations")
        span.set_status(Status(StatusCode.ERROR, "loop hit iteration cap"))
```

A run that terminates on `max_iterations` instead of `final` is a *latent runaway* — it produced an answer but only because you cut it off. That distinction is invisible unless you record it, and it's one of the most common silent cost incidents in production agents.

## Problem 2 — Sessions that outlive a single run

A user has a conversation; a workflow agent runs for hours across many invocations. These are **sessions**, and they are a level *above* the trace. The GenAI conventions and every major platform support this with a session identifier you propagate onto every trace in the conversation:

```python
attributes = {
    "session.id": session_id,        # stable across the whole conversation
    "gen_ai.conversation.id": session_id,
    "user.id": hashed_user_id,       # hashed, never raw PII
}
```

Three nesting levels, each answering a different operational question:

```text
session.id = S  ──────────────────────────────  "this user's whole conversation"
├── trace_id = T1  invoke_agent (turn 1)  ─────  "one run / one turn"
│   ├── chat
│   └── execute_tool
├── trace_id = T2  invoke_agent (turn 2)
└── trace_id = T3  invoke_agent (turn 3)
```

The session view is where multi-turn failures live: an agent that "forgets" a constraint the user gave three turns ago is a *memory* failure (see [mod-303](../mod-303-memory-context-architecture/README.md)) that no single trace reveals. You can only see it by reading T3 against T1 — which requires `session.id` on both. Decide your session boundary deliberately: too coarse (a session per user, forever) and traces become un-navigable; too fine (a session per turn) and you lose the multi-turn view entirely.

## Problem 3 — Comparability across deploys

Because every run differs, you cannot judge a deploy by eyeballing one trace. You judge it by comparing **distributions** of a quality signal before and after — which only works if every span is stamped with what produced it. This is the payoff for the `Resource` attributes from Chapter 1, plus the prompt and model versions:

| Attribute | Carried where | Answers |
|---|---|---|
| `service.version` | Resource (all spans) | "which code build produced this run?" |
| `gen_ai.request.model` | every `chat` span | "which exact model?" |
| `prompt.version` | every `chat` span | "which prompt template revision?" |
| `deployment.environment` | Resource (all spans) | "prod, staging, or eval?" |

With these, the deploy question becomes a query: *group faithfulness by `service.version`, compare the two distributions.* A step-change down is a regression you can attribute and roll back. Without `prompt.version`, a prompt edit that tanks quality is indistinguishable from random run-to-run variance — you'll see the score drop and have nothing to blame.

```text
faithfulness distribution, grouped by deploy
v2026.05:  mean 0.91  ████████████████
v2026.06:  mean 0.86  ████████████        ← attributable regression, not noise
```

## Problem 4 — Sampling when every run is different and large

You cannot afford to store full prompt/completion content for 100% of high-volume traffic, and you cannot afford to run an LLM-judge on every trace. But you also can't blindly sample 1% and miss every failure, because **the failures are the rare runs you most need to keep.** The design is tail-biased, not uniform:

- **Keep all errors and all guardrail trips.** A `1%` uniform sample throws away 99% of your incidents. Sample errors at 100%.
- **Keep low-quality and high-cost runs.** A run that scored 0.3 on the rubric or burned 50k tokens is worth more than a hundred clean runs. Bias retention toward the tails.
- **Use tail sampling, not head sampling.** Head sampling decides at the start of a trace, before you know if it failed. **Tail sampling** decides after the trace completes, when the error status, the quality score, and the cost are all known — exactly the signals you want to sample *on*. It costs more (you buffer the trace) and it's the right default for agents.
- **Capture content selectively.** Store full prompts/completions for the sampled-and-kept traces (debugging needs them); store attributes-only for the rest (trends only need the numbers).

```text
head sampling:  decide at span start  →  blind to outcome  →  drops failures
tail sampling:  decide at trace end    →  sees error+score+cost  →  keeps failures ✓
```

## Key takeaways

- **One logical run = one trace; retries and loop iterations are spans under it** — carry `retry.attempt` and `agent.loop.iteration`/`terminated` so a runaway loop and a legitimate multi-tool run are distinguishable.
- **Sessions are a level above traces** — propagate `session.id` onto every trace in a conversation so multi-turn and memory failures become visible.
- **Comparability across deploys is a query, not an eyeball** — stamp `service.version`, `gen_ai.request.model`, and `prompt.version` on spans so a regression is attributable and a drift has a cause.
- **Sample on the tail, not the head** — decide retention after the trace completes so you keep the rare failed, low-quality, and high-cost runs that uniform sampling would discard.
