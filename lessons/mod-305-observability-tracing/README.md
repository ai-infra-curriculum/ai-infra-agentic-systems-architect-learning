# mod-305-observability-tracing: Observability & Tracing for Agents

**Estimated effort:** 10 hours

You cannot operate what you cannot see. A multi-agent system in production fails in ways a dashboard of CPU and p99 latency will never surface: a worker quietly hallucinates a citation, a tool-selection prompt drifts after a model upgrade, an orchestrator loops three extra times because a retrieval call returned empty. Traditional APM tells you the request was *fast and returned 200*. It says nothing about whether the answer was *right*. This module is where you architect the observability layer that closes that gap — instrumenting agents with open, vendor-neutral conventions, defining the quality and drift signals that matter for non-deterministic systems, and choosing a platform you won't have to rip out in six months.

> **Architect, don't bolt on.** Observability for agents is a design decision made *before* the first deploy, not a dashboard you add after the first incident. You'll design the span schema, the signal taxonomy, and the platform-evaluation criteria as first-class architecture artifacts — the same way you'd design a data model or an API contract.

## Learning objectives

- **Instrument agents with OpenTelemetry GenAI semantic conventions** — `invoke_agent` and `execute_tool` spans, token and model attributes, and the span hierarchy that makes a trace readable.
- **Distinguish agent observability from traditional APM** — output quality, faithfulness, safety, and drift are the signals that matter, and none of them are latency or error rate.
- **Architect tracing for non-deterministic, long-running agents** — stable trace identity across retries, sessions that span hours, and comparability across model and prompt deploys.
- **Evaluate observability platforms** — LangSmith, Langfuse, and Arize Phoenix against a requirements matrix you derive from your own system, not a feature checklist.

## Lecture chapters

1. [Instrumenting Agents with OpenTelemetry GenAI](01-otel-genai-instrumentation.md) — the GenAI semantic conventions, `invoke_agent` / `execute_tool` spans, token attributes, and a working OTel SDK setup.
2. [Agent Observability vs. Traditional APM](02-observability-vs-apm.md) — why faithfulness, quality, safety, and drift replace latency and error rate as your golden signals.
3. [Tracing Non-Deterministic, Long-Running Agents](03-tracing-nondeterministic-agents.md) — trace identity across retries, sessions across deploys, and sampling when every run is different.
4. [Evaluating Observability Platforms](04-platform-evaluation.md) — a requirements-driven evaluation of LangSmith, Langfuse, and Arize Phoenix, and the build-vs-buy decision.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

- [exercise-01: OTel GenAI span design](exercises/exercise-01-otel-genai-span-design.md) — design and implement the span schema for a multi-agent run using GenAI semantic conventions.
- [exercise-02: Quality and drift signals](exercises/exercise-02-quality-and-drift-signals.md) — define the non-APM signal taxonomy and wire faithfulness and drift detection into traces.
- [exercise-03: Observability platform evaluation](exercises/exercise-03-observability-platform-evaluation.md) — build a weighted evaluation matrix and run a real proof-of-concept against two platforms.

## Quiz

- [Knowledge check](quizzes/quiz-01.md) — span conventions, agent-vs-APM signals, and platform trade-offs.

## Prerequisites

- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — the orchestrator-worker and handoff topologies you'll be tracing.
- [mod-304: Evaluation Harnesses](../mod-304-evaluation-harnesses/README.md) — the LLM-as-judge rubrics you'll attach to spans as quality signals.
- Working OpenTelemetry knowledge (spans, traces, context propagation) or willingness to pick it up from Chapter 1.
- Comfort with `async` Python — the instrumentation examples use the OTel Python SDK.

See [resources.md](resources.md) for primary references.
