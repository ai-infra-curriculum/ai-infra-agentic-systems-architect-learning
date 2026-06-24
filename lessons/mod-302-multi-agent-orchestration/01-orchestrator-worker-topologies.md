# Chapter 1 — Orchestrator-Worker Topologies and Token Economics

An orchestrator-worker system has a **lead** agent that decomposes a task, fans it out to **workers** that each run in their own context, and **synthesizes** their returns. The engineer's question is *how do I wire the loop*. The architect's question is *what shape does the fan-out take, and can the budget afford it*. This chapter is about the second question.

```text
                ┌──────────────┐
   task ───────▶│ orchestrator │  decompose (bounded to N)
                └──────┬───────┘
            ┌──────────┼──────────┐
            ▼          ▼          ▼
        ┌───────┐  ┌───────┐  ┌───────┐
        │worker │  │worker │  │worker │   isolated context each
        └───┬───┘  └───┬───┘  └───┬───┘
            └──────────┼──────────┘
                       ▼
                ┌──────────────┐
                │ orchestrator │  synthesize ──▶ result
                └──────────────┘
```

## Why ~15x: where the tokens go

Anthropic's multi-agent research system reports that agents use roughly **4x the tokens of a chat interaction**, and a *multi-agent* system uses roughly **15x**. That number is not a tax you can optimize away — it is structural, and understanding *where* it comes from tells you when to spend it.

- **Per-worker context duplication.** The orchestrator's framing, the shared task context, and tool schemas are re-paid in *every* worker's context window. Five workers means five copies of the boilerplate.
- **Orchestrator round-trips.** Decompose is one model call; synthesize is another; gap-filling follow-ups add more. The lead pays for the whole conversation, twice (in and out).
- **Wide tool surfaces.** Each worker that can call tools re-loads tool definitions into its context on every turn.

The payoff: workers run **in parallel** (wall-clock collapses toward the slowest worker, not the sum) and each keeps a **clean, isolated context** so breadth that would overflow a single window becomes tractable. You are trading tokens for breadth and latency.

## Topology shapes and what they cost

| Topology | Shape | Token profile | Best for |
| --- | --- | --- | --- |
| Single agent | One loop | 1x baseline | Few tool calls, narrow task |
| Sequential chain | A → B → C | ~2–4x; no parallelism | Dependent steps, validation gates |
| Orchestrator + N workers | Fan-out → synth | ~10–20x; parallel | Independent, broad work |
| Hierarchical | Orchestrator → sub-orchestrators → workers | 20x+; deep | Very large decompositions |

Hierarchy multiplies the multiplier: a sub-orchestrator is itself a ~15x system nested inside a worker slot. Reach for it only when a single decomposition layer genuinely can't express the task, because the cost compounds.

## Sizing a topology to its budget

Treat the budget as a design constraint, not an afterthought. A rough envelope:

```text
est_tokens ≈ orchestrator_overhead
           + N_workers × (shared_context + worker_work + distilled_return)
           + synthesis_pass

where distilled_return ≪ worker_work   (this ratio is the whole game)
```

The lever you control most directly is the **distilled return**: workers hand back a summary, not their transcript, so the orchestrator's context grows by *summaries × N*, not *transcripts × N*. If returns aren't distilled, the orchestrator's context — and cost — explodes superlinearly.

## Design rules that govern the economics

- **Bound the fan-out (N ≤ some cap).** An unbounded decomposition that spawns 200 workers is a cost incident, not a feature. The cap is an architectural decision; surface it in the spec.
- **Decompose only what's independent.** If sub-tasks depend on each other in sequence, fan-out wastes tokens re-establishing context — that's a chain (Chapter 2 / handoffs), not a fan-out.
- **Insist on distilled returns.** This is the single biggest lever on orchestrator-context growth and therefore on cost.
- **Match the multiplier to the value.** A ~15x spend is justified when breadth or parallel latency is worth real money; for a task a single agent handles in three tool calls, it is pure waste.

## Key takeaways

- A multi-agent system costs ~15x a single chat (vs. ~4x for a lone agent); the cost is structural — context duplication, orchestrator round-trips, and wide tool surfaces.
- Topology choice is a cost decision: single < chain < fan-out < hierarchy, and hierarchy compounds the multiplier.
- The distilled-return ratio is the dominant lever on orchestrator-context growth — design for it explicitly.
- Bound the fan-out, decompose only independent work, and only spend the 15x when breadth or parallel latency earns it.
