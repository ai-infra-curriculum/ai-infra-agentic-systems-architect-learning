# mod-304-evaluation-harnesses: Evaluation Harnesses for Agentic Systems

**Estimated effort:** 12 hours

A demo that works once is not a system you can deploy. Agentic systems fail in ways unit tests never catch: the agent reaches the right answer through the wrong sequence of tool calls, calls the correct tool with malformed arguments, cites a source it never retrieved, or burns ten tool calls on a task that needed two. As the architect, your job is not to hand-check transcripts — it is to design the **evaluation harness** that decides, automatically and repeatably, whether a change is safe to ship.

This module treats evaluation as a **deployment gate**, not an afterthought. You'll design two complementary lenses — **trajectory evaluation** (was the *path* correct?) and **final-state evaluation** (was the *outcome* correct?) — then build **tool-call correctness** checks at the selection and argument layers, author **LLM-as-judge rubrics** with the discipline of a measurement instrument, and wire the whole thing into an **offline/online strategy** that blocks a bad release before it reaches users.

> **Eval is architecture.** The eval harness is a load-bearing component, on the critical path of every deploy. Design it with the same rigor you give the agent itself: typed contracts, versioned datasets, deterministic-where-possible scoring, and a clear pass/fail gate. A harness you don't trust is worse than none — it gives false confidence.

## Learning objectives

- Design **trajectory evaluation** (the sequence of messages and tool calls) and **final-state evaluation** (the end artifact or world state), and decide which lens each requirement needs.
- Build **tool-call correctness** checks at two layers — **selection** (did the agent pick the right tool?) and **arguments** (did it call it correctly?) — combining deterministic assertions with an LLM judge where determinism runs out.
- Author **LLM-as-judge rubrics** that score accuracy, completeness, citation faithfulness, and tool efficiency reliably enough to gate on.
- Define an **offline/online eval strategy** that gates deployment: a regression suite in CI, canary/online checks in production, and explicit promotion criteria.

## Lecture chapters

1. [Trajectory vs. Final-State Evaluation](01-trajectory-vs-final-state-eval.md) — the two lenses, what each catches that the other misses, and how to design datasets for both.
2. [Tool-Call Correctness: Selection and Argument Layers](02-tool-call-correctness.md) — deterministic checks for the selection and argument layers, and where an LLM judge takes over.
3. [LLM-as-Judge Rubrics](03-llm-as-judge-rubrics.md) — authoring, calibrating, and bias-proofing rubrics for accuracy, completeness, citation, and tool efficiency.
4. [Offline/Online Eval Strategy as a Deployment Gate](04-eval-strategy-deployment-gate.md) — the regression suite, the online canary, promotion criteria, and the pipeline that ties them together.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-engineering-curriculum/agentic-systems-architect-solutions).

- [exercise-01: Design a trajectory eval](exercises/exercise-01-trajectory-eval-design.md) — specify a trajectory-vs-final-state dataset and scorer for a multi-step agent.
- [exercise-02: Build a tool-call correctness harness](exercises/exercise-02-tool-call-correctness-harness.md) — selection and argument checks, deterministic plus LLM-judge.
- [exercise-03: Author an LLM-as-judge rubric](exercises/exercise-03-llm-as-judge-rubric.md) — write and calibrate a four-axis rubric against a human-labeled gold set.
- [exercise-04: Wire an eval-gated release pipeline](exercises/exercise-04-eval-gated-release-pipeline.md) — design the offline + online gate that blocks a regressing build.

## Quiz

- [Evaluation harness knowledge check](quizzes/README.md) — trajectory vs. final-state, correctness layers, judge calibration, and gating strategy.

## Prerequisites

- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — you're evaluating the trajectories these systems produce.
- [mod-305: Observability & Tracing](../mod-305-observability-tracing/README.md) — pairs with this module; traces are the raw material trajectory eval scores. Helpful but not required.
- Comfort reading a serialized agent trace (messages + tool calls) and writing Python.

See [resources.md](resources.md) for primary references.
