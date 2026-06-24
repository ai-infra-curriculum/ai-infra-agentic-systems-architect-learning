# exercise-01: OTel GenAI Span Design

**Estimated effort:** 3 hours

## Objective

Design and implement the span schema for a multi-agent run using the OpenTelemetry GenAI semantic conventions. You'll produce a **span-schema design artifact** first (the architect's deliverable), then instrument a small orchestrator-worker system to emit a trace that matches it — `invoke_agent`, `chat`, and `execute_tool` spans, correct attributes, correct nesting, and failures recorded as red spans. By the end you'll be able to read a multi-agent trace and trust that its hierarchy reflects the run.

## Background

This exercise covers material from:

- [Chapter 1 — Instrumenting Agents with OpenTelemetry GenAI](../01-otel-genai-instrumentation.md)
- [Chapter 3 — Tracing Non-Deterministic, Long-Running Agents](../03-tracing-nondeterministic-agents.md) (loop and retry attributes)

Use any model provider and any small orchestrator-worker agent (one orchestrator, two workers, at least one tool is enough). You may reuse a system from [mod-302](../../mod-302-multi-agent-orchestration/README.md). For the backend, anything that speaks OTLP works — a local Arize Phoenix or Langfuse instance, or even the console exporter to start.

## Prerequisites

- Python with `opentelemetry-sdk`, `opentelemetry-exporter-otlp`, and `opentelemetry-semantic-conventions` installed (pin the semconv version).
- A small orchestrator-worker agent with at least one tool call.
- An OTLP-capable viewer (local Phoenix/Langfuse) or the console span exporter.

## Tasks

### 1. The span-schema design artifact

Before writing code, write `SPAN_SCHEMA.md`. For each span type your run will emit (`invoke_agent`, `chat`, `execute_tool`), specify:

- The exact **span name** pattern (e.g., `invoke_agent {agent_name}`).
- The **required attributes** with their GenAI-convention keys (`gen_ai.operation.name`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`, etc.).
- The **parent** each span attaches to, i.e. the intended hierarchy.
- Which data is a **span attribute** vs. a **span event** (prompts/completions are events) vs. **dropped** (and your content-capture decision, with the PII reasoning).

This artifact is the contract the code must satisfy. Cite the convention version you targeted.

### 2. Process-level setup

- Configure a `TracerProvider` with a `Resource` carrying `service.name`, `service.version`, and `deployment.environment`.
- Wire a `BatchSpanProcessor` and an OTLP exporter (or console exporter).

### 3. Instrument the run to match the schema

- Open an `invoke_agent` root span for the orchestrator; open child `invoke_agent` spans for each worker.
- Inside each worker, emit `chat` spans (with `gen_ai.request.model` and `gen_ai.usage.*` tokens) and `execute_tool` spans (with `gen_ai.tool.name`).
- Rely on **context propagation** for nesting — do not pass span objects by hand. Verify the emitted tree matches the hierarchy in your artifact.

### 4. Record a failure

- Make one tool call or one worker fail. On its span, set `Status(StatusCode.ERROR, ...)` and call `record_exception`. Confirm the failed span renders red **in the context of the surrounding run**, not as an orphan.

### 5. Loop and retry attributes

- Add `agent.loop.iteration` to the worker span and `agent.loop.terminated` (`final` vs. `max_iterations`).
- On a retried `chat` call, add `retry.attempt`. Demonstrate a trace where three retries of one call are visibly distinct from three legitimate separate tool calls.

## Starter guidance

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.resources import Resource
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.trace import Status, StatusCode

GEN_AI = "gen_ai"

provider = TracerProvider(resource=Resource.create({
    "service.name": "research-agent",
    "service.version": "2026.06.0",
    "deployment.environment": "dev",
}))
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("agent.runtime")


async def run_worker(role: str, instruction: str):
    with tracer.start_as_current_span(
        f"invoke_agent {role}",
        attributes={f"{GEN_AI}.operation.name": "invoke_agent",
                    f"{GEN_AI}.agent.name": role},
    ) as span:
        # chat + execute_tool spans created in here nest automatically
        ...  # set gen_ai.usage.* tokens on the chat spans
        # on failure: span.set_status(Status(StatusCode.ERROR, msg)); span.record_exception(exc)
        ...
```

You do **not** need the quality/drift signals (exercise-02) or a platform comparison (exercise-03) here.

## Acceptance criteria

You can demonstrate that:

- `SPAN_SCHEMA.md` exists and names every span type, its convention attributes, its parent, and the attribute-vs-event-vs-dropped decision with a content-capture rationale.
- The emitted trace is **one trace** with the orchestrator `invoke_agent` as root and worker `invoke_agent` spans as children; `chat`/`execute_tool` spans nest under their worker via context propagation, not manual wiring.
- `chat` spans carry `gen_ai.request.model` and both token attributes; `execute_tool` spans carry `gen_ai.tool.name`.
- A failed span is **red and in-context**, with a recorded exception.
- A retried call is distinguishable from repeated legitimate tool calls via `retry.attempt`; a loop that hit its cap shows `agent.loop.terminated = max_iterations`.

## Reflection

In `NOTES.md`:

1. Where did your *emitted* hierarchy diverge from your *designed* hierarchy on the first run, and what caused it (usually a span opened without being made current)?
2. What did you decide about capturing prompt/completion content, and what would change that decision in production?
3. Which convention attribute names had changed or were marked experimental in the version you pinned?

## Stretch goals

- Add `session.id` and re-run the agent for two turns; confirm both traces group under one session in your viewer.
- Swap the OTLP backend (console → Phoenix, or Phoenix → Langfuse) **without touching instrumentation code** — proving the convention bought you portability.
- Add an OTel `Counter` for total tokens and confirm metrics and traces share the same `service.version`.
