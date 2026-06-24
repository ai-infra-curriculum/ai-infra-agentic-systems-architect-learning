# Chapter 2 — Agent Observability vs. Traditional APM

Application Performance Monitoring (APM) answers a question with a known shape: *was the request fast, did it error, how many per second?* The golden signals — latency, traffic, errors, saturation — assume the system is **deterministic**: the same input produces the same output, so a correct response is a `200` returned quickly. Agents break that assumption at the root. The same input produces a *different* output every run, and a `200` returned in 800ms can contain a fabricated citation, a leaked secret, or a confidently wrong number. APM will report that run as a success. It is the single most dangerous blind spot in operating agents.

Agent observability does not *replace* APM — you still need latency and error rate to keep the service up. It **adds a second plane of signals about output quality** that APM has no vocabulary for. As the architect, your job is to specify both planes and to be explicit that the quality plane is the one that catches the failures users actually feel.

## The two planes, side by side

```text
                APM plane                    Agent-quality plane
   ─────────────────────────────   ──────────────────────────────────────
   latency (p50/p95/p99)           faithfulness  (grounded in sources?)
   error rate (5xx)                output quality (rubric / LLM-judge score)
   throughput (req/s)              safety        (PII leak, jailbreak, toxicity)
   saturation (CPU/mem/queue)      task success  (did it accomplish the goal?)
   ─────────────────────────────   ──────────────────────────────────────
   "is the service up & fast?"     "are the answers right & safe?"
   deterministic, request-scoped   non-deterministic, trajectory-scoped
```

The left plane is necessary and not sufficient. A system can be 100% green on the left and failing every user on the right.

## The four signals that matter

These are the signals APM cannot produce. Each attaches to a span or a trace as an attribute or an evaluation result — instrument them and they become as queryable as latency.

**1. Faithfulness (groundedness).** Is every claim in the output supported by the retrieved context or tool results? A RAG agent that cites a source it never read, or a research worker that adds a "fact" not present in any worker result, is *unfaithful* — fluent and plausible and wrong. You measure it by comparing the output against the span's retrieved context, typically with an LLM-judge (the rubrics from [mod-304](../mod-304-evaluation-harnesses/README.md)) or an NLI model. Faithfulness is the single highest-value agent signal because hallucination is the failure mode users trust least once they catch it.

**2. Output quality.** Did the answer actually satisfy the request — complete, relevant, correct, well-cited? This is the rubric score from your eval harness, attached to the trace. It is inherently graded (a 0.7, not a pass/fail) and inherently subjective, which is exactly why APM has no slot for it.

**3. Safety.** Did the run leak PII, emit toxic content, comply with a jailbreak, or call a tool it should have been denied? Safety signals are *guardrail outcomes* (you'll build the guardrails in [mod-306](../mod-306-guardrails-safety-security/README.md)) recorded onto the trace. A safety regression is invisible to APM and catastrophic to ship.

**4. Task success.** Over the whole trajectory — not one request — did the agent accomplish the user's goal? A booking agent that returns a beautifully formatted confirmation for the *wrong date* is a `200` and a task failure. This signal is trajectory-scoped: it can only be judged against the full span tree, which is why it lives in your tracing layer and not in a per-request metric.

## Why APM tooling can't be repurposed

It is tempting to think you can bolt these onto an existing APM stack. Four structural mismatches say otherwise:

- **No ground truth at request time.** APM knows `500` is bad with zero context. "Is this answer faithful?" requires the retrieved context, the prompt, *and* a judgment call — none of which a metric pipeline carries.
- **The unit of analysis is the trajectory, not the request.** Quality emerges from a *sequence* of spans (retrieve → reason → tool → synthesize). APM aggregates per-request and discards the sequence.
- **Outputs are unstructured and high-cardinality.** You cannot histogram free-text answers. You score them, and scoring is itself an LLM call with cost and latency.
- **Evaluation is asynchronous and sampled.** You can't judge faithfulness inline without doubling latency. Quality signals are computed *after* the trace lands, on a sample, and joined back by `trace_id` — a workflow APM was never built for.

## Drift: the signal that only exists over time

The fifth signal is not per-trace at all — it is the *trend*. **Drift** is the slow degradation of behavior while every individual run still looks fine:

- **Input drift** — users start asking questions your agent was never tuned for; the topic distribution shifts.
- **Output/quality drift** — average rubric score sags 0.05 a week after a retrieval-index change nobody connected to it.
- **Tool-use drift** — a model upgrade subtly changes how often the agent picks the right tool; selection accuracy slides.
- **Cost drift** — average tokens-per-run creeps up as prompts accrete, and your unit economics quietly invert.

Drift is invisible at the single-trace level *by definition* — each run passes. You catch it only by aggregating a quality signal over a time window and alerting on the *slope*, segmented by `service.version` so you can attribute the regression to a deploy. This is why Chapter 3's deploy-aware trace identity is a hard requirement and not a nicety: without `service.version` and `gen_ai.request.model` on every span, you can see the drift but never name its cause.

```text
quality score over time (segmented by deploy)
1.0 ┤████████████ v2026.05
0.9 ┤            ████████ v2026.06   ← step-down after deploy: a regression
0.8 ┤                    ▓▓▓▓▓▓▓▓    ← slow slope: drift, no deploy to blame
    └──────────────────────────────▶ time
```

## Key takeaways

- APM's golden signals assume **determinism**; agents are non-deterministic, so a fast `200` can still be a wrong, unsafe, or off-goal answer that APM scores as success.
- Agent observability **adds a quality plane** — faithfulness, output quality, safety, task success — on top of APM, not in place of it.
- These signals are **trajectory-scoped, ground-truth-free, and computed asynchronously on a sample**, which is structurally why APM pipelines can't produce them.
- **Drift** is a trend-only signal — invisible per-trace — that you catch by aggregating quality over time and segmenting by `service.version` and model, which makes deploy-aware trace identity a requirement.
