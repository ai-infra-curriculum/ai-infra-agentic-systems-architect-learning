# exercise-03: Observability Platform Evaluation

**Estimated effort:** 3 hours

## Objective

Make a defensible build-vs-buy decision for an agent observability platform. You'll derive weighted requirements from a concrete system, build a **weighted scoring matrix**, run a real proof-of-concept ingesting your own traces into **two** of {LangSmith, Langfuse, Arize Phoenix}, and write a one-page **architecture decision record (ADR)** recommending one. The deliverable is the decision artifact an architect defends in review — not a vibe.

## Background

This exercise covers material from:

- [Chapter 4 — Evaluating Observability Platforms](../04-platform-evaluation.md)
- [Chapter 2](../02-observability-vs-apm.md) and [Chapter 3](../03-tracing-nondeterministic-agents.md) — the eval-integration and deploy-comparison capabilities your POC must actually exercise.

Use the instrumented, signal-emitting agent from [exercise-01](exercise-01-otel-genai-span-design.md) and [exercise-02](exercise-02-quality-and-drift-signals.md) as the system under evaluation. At least two platforms have free local/self-host or free-tier options (Langfuse and Arize Phoenix self-host trivially; LangSmith has a free tier), so a real POC is achievable at zero cost.

## Prerequisites

- The instrumented agent emitting GenAI-convention OTLP traces with quality scores (exercises 01–02).
- Accounts or local instances for **two** platforms you'll actually test.
- A small batch of real-or-realistic traces (≥20 runs) to ingest.

## Tasks

### 1. Requirements, weighted to *your* system

Write `REQUIREMENTS.md`. State the system under evaluation in three sentences (topology, data sensitivity, scale). Then list the evaluation criteria — start from Chapter 4 (OTel-native ingestion, self-host vs. managed, eval integration, cost model at scale, datasets/experiments, framework coupling, operational maturity) — and **assign each a weight (1–5)** justified by *this* system. A healthcare agent weights self-host at 5; a hackathon prototype weights it at 1. The weights are the architecture.

### 2. The proof-of-concept

For **two** platforms, actually:

- **Ingest** your OTLP traces and confirm the GenAI-convention spans render with the right hierarchy and attributes (does it understand `invoke_agent`/`execute_tool`, or flatten them?).
- **Run one eval** on captured traces (an LLM-judge faithfulness score) and confirm it joins by `trace_id`.
- **Exercise the deploy comparison** from Chapter 3 — can you group a quality signal by `service.version` and see the regression?
- **Curate one dataset** of failing traces into a regression set.

Record what worked, what was painful, and what was impossible — per platform.

### 3. The scoring matrix

Build the matrix: criteria (rows) × platforms (columns), each cell a 1–5 score from the **POC, not the marketing page**. Compute weighted totals.

```text
criterion (weight)        | Langfuse | Phoenix | (LangSmith)
--------------------------|----------|---------|-----------
OTel-native ingest (5)    |    5     |    5    |    4
self-host (5)             |    5     |    5    |    2
eval integration (4)      |    4     |    5    |    5
cost at scale (3)         |    5     |    5    |    3
datasets/experiments (3)  |    4     |    4    |    5
framework coupling (2)    |    5     |    5    |    4
ops maturity (3)          |    3     |    3    |    5
--------------------------|----------|---------|-----------
weighted total            |   ...    |   ...   |    ...
```

### 4. The ADR

Write `ADR-observability-platform.md` (one page): context, the options considered, the decision, the **consequences and what you're giving up**, and the explicit condition that would make you revisit (e.g., "if trace volume exceeds X/month, re-model cost"). An honest ADR names the losing trade-offs.

## Acceptance criteria

You can demonstrate that:

- `REQUIREMENTS.md` states the system and assigns **justified weights** tied to that system's data sensitivity and scale.
- The POC was **real**: your own traces ingested into two platforms, with notes on hierarchy fidelity, eval join, deploy comparison, and dataset curation per platform.
- The scoring matrix uses **POC-derived** scores and computes weighted totals; the recommendation follows from the numbers.
- The ADR names the decision, the **trade-offs given up**, and a concrete revisit condition.

## Reflection

In `NOTES.md`:

1. Which platform *demoed* best vs. *scored* best on your weighted matrix? Where did the demo and the POC disagree?
2. Did any platform fail to render your GenAI-convention spans faithfully? What did that cost in debuggability?
3. Under what change to your system (scale, data sensitivity, framework) would your recommendation flip — and is that the revisit condition in your ADR?

## Stretch goals

- Add the **build** option as a third column: estimate the engineering cost of OTel Collector → ClickHouse → Grafana **plus** the eval/dataset product, and score it honestly against buy.
- Model the **cost curve**: plot each platform's annual cost against trace volume from 10k to 10M traces/month and find the crossover points.
- Run the *same* eval in both platforms and confirm the faithfulness scores agree — if they don't, figure out which judge config differs.
