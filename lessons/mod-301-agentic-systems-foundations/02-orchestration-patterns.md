# Chapter 2 — The Orchestration Pattern Catalog

Anthropic's *Building effective agents* names five composable building blocks. Four are **workflows** (fixed control flow); one — the autonomous agent loop — is the agent itself. An architect should be able to look at a task and name which pattern fits, the way a structural engineer looks at a span and names the truss. This chapter is your catalog: the shape of each pattern, the task it fits, and what it costs.

The patterns compose. A real system is usually a router on the outside, a chain in one branch, and an orchestrator-workers block in another. Learn them as primitives.

## 1. Prompt chaining

Decompose a task into a fixed sequence of LLM calls, each operating on the previous output. Add programmatic gates between steps to catch failures early.

```text
 in ─▶[ LLM 1 ]─▶( gate )─▶[ LLM 2 ]─▶[ LLM 3 ]─▶ out
                    │
                    └─ fail ─▶ stop / fix
```

- **Fits:** tasks that cleanly split into fixed subtasks ("draft, then translate, then tighten"). Trades latency for accuracy by making each call easier.
- **Cost:** linear in steps; fully predictable; trivially debuggable (each step replays).
- **Architect's note:** the gates are the value. A chain without validation between steps is just a long prompt.

## 2. Routing

Classify the input, then dispatch it to one of several specialized handlers. The classifier is an LLM call (or a cheap model); the handlers can be anything.

```text
              ┌─▶ [ handler A ]
 in ─▶[ route ]─▶ [ handler B ] ─▶ out
              └─▶ [ handler C ]
```

- **Fits:** distinct input categories that each deserve different handling (refund vs. technical vs. billing support; easy queries to a small model, hard ones to a large one).
- **Cost:** one classification call plus one handler; predictable; lets you optimize each path independently.
- **Architect's note:** routing is how you keep prompts focused. One catch-all prompt that tries to handle every category degrades on all of them.

## 3. Parallelization

Run multiple LLM calls at once, then aggregate. Two flavors:

- **Sectioning:** split the work into independent subtasks that run concurrently (review code for security *and* style *and* performance in parallel).
- **Voting:** run the *same* task several times for diverse outputs, then aggregate (majority vote, or "flag if any run says unsafe").

```text
        ┌─▶ [ LLM ]─┐
 in ───▶┼─▶ [ LLM ]─┼─▶ ( aggregate ) ─▶ out
        └─▶ [ LLM ]─┘
```

- **Fits:** independent subtasks (sectioning) or confidence/coverage via repetition (voting).
- **Cost:** more tokens, but latency stays near the slowest branch instead of the sum.
- **Architect's note:** sectioning also improves *quality* — each call has a narrow focus and a clean context, not one overloaded prompt.

## 4. Orchestrator-workers

A lead LLM dynamically decomposes a task, dispatches subtasks to worker LLMs, and synthesizes their results. Unlike parallelization, **the subtasks are not predetermined** — the orchestrator decides them at runtime.

```text
                 ┌─────────────┐
 task ─────────▶ │ orchestrator│  decompose (dynamic)
                 └──────┬──────┘
            ┌───────────┼───────────┐
            ▼           ▼           ▼
        [ worker ]  [ worker ]  [ worker ]
            └───────────┼───────────┘
                        ▼
                 ┌─────────────┐
                 │ orchestrator│  synthesize ─▶ result
                 └─────────────┘
```

- **Fits:** complex tasks where you cannot predict the subtasks in advance (multi-file code changes, open-ended research where the orchestrator decides which threads to pull).
- **Cost:** high — this is where token spend multiplies (a multi-agent research system can run many times the tokens of a single chat). It is the first pattern that is genuinely **agentic** in its decomposition step.
- **Architect's note:** because decomposition is dynamic, you must **bound the fan-out** or accept unbounded cost.

## 5. Evaluator-optimizer

One LLM generates a candidate; a second LLM evaluates it against criteria and returns feedback; the generator revises. Loop until the evaluator passes the output or a cap is hit.

```text
 in ─▶[ generator ]─▶ candidate ─▶[ evaluator ]
            ▲                            │
            └────── feedback ────────────┘
                                         │ pass / cap
                                         ▼
                                        out
```

- **Fits:** tasks with clear evaluation criteria where iteration measurably helps (literary translation, complex search refinement, "keep improving until it clears the bar").
- **Cost:** variable — depends on iterations; **must have a hard iteration cap** or it can loop indefinitely.
- **Architect's note:** only worth it when you can articulate the criteria. If you cannot write the rubric, the evaluator has nothing to check against.

## Pattern selection table

| Task shape | Pattern | Why |
| --- | --- | --- |
| Fixed sequence of subtasks | Prompt chaining | Predictable, each step validated |
| Distinct input categories | Routing | Specialize each path, keep prompts focused |
| Independent subtasks, run together | Parallelization (sectioning) | Concurrency + narrow context per call |
| Need confidence or coverage | Parallelization (voting) | Diverse runs aggregated |
| Subtasks unknown until runtime | Orchestrator-workers | Dynamic decomposition |
| Clear criteria + iteration helps | Evaluator-optimizer | Critic-driven refinement to a bar |

## Cost and predictability at a glance

| Pattern | Control flow | Relative cost | Predictability |
| --- | --- | --- | --- |
| Prompt chaining | Fixed | Low | High |
| Routing | Fixed | Low | High |
| Parallelization | Fixed | Medium | High |
| Orchestrator-workers | Model-driven decomposition | High | Medium |
| Evaluator-optimizer | Looping, capped | Variable | Medium |

The first three are pure workflows: you can enumerate every path. Orchestrator-workers and evaluator-optimizer introduce model-driven control flow and the variance that comes with it. That boundary — fixed-path vs. model-driven — is the same line Chapter 1 drew, now visible inside the catalog.

## Key takeaways

- Five patterns, four of them workflows: **chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer**.
- Patterns **compose** — real systems nest them; learn them as primitives and combine.
- The two with model-driven control flow (orchestrator-workers, evaluator-optimizer) carry the cost and variance; the other three are predictable.
- Match the **task shape** to the pattern; do not default to the most powerful one because it is the most impressive.
