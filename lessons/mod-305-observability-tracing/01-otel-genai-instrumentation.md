# Chapter 1 — Instrumenting Agents with OpenTelemetry GenAI

OpenTelemetry (OTel) is the vendor-neutral standard for traces, metrics, and logs. Its **GenAI semantic conventions** extend the standard with a shared vocabulary for LLM and agent operations: a registry of span names, attribute keys, and event shapes so that *any* backend — LangSmith, Langfuse, Arize Phoenix, Datadog, your own ClickHouse — reads the same trace the same way. As the architect, this is the decision that buys you portability: instrument once against the convention, and your backend becomes a swappable component instead of a lock-in.

The conventions are still marked **experimental** in the OTel spec — attribute names have changed across releases (the namespace moved from `gen_ai.*` toward `gen_ai.*` with operation-specific refinements, and `invoke_agent` / `execute_tool` are recent additions). Pin your `opentelemetry-semantic-conventions` version and treat the names as a contract you re-verify on upgrade.

## The trace hierarchy of an agent run

A single agent run is a **trace** — one tree of **spans**, each span a timed operation with attributes and a parent. The GenAI conventions define the span *operations* that matter for agents. A well-formed orchestrator-worker run looks like this:

```text
invoke_agent  orchestrator                    (root span, trace_id = T)
├── chat      gpt-4o  (decompose)             gen_ai.operation.name = chat
├── invoke_agent  worker[research]            (child agent, same trace)
│   ├── chat      gpt-4o
│   ├── execute_tool  web_search              gen_ai.operation.name = execute_tool
│   └── chat      gpt-4o  (distill)
├── invoke_agent  worker[numbers]
│   ├── execute_tool  calculator
│   └── chat      gpt-4o
└── chat      gpt-4o  (synthesize)
```

Three operation types carry the agent semantics:

- **`invoke_agent`** — one agent's turn or full invocation. Span name is `invoke_agent {agent_name}`. This is the unit a human reads to ask "what did the research worker actually do?"
- **`chat`** (or `text_completion`) — one model inference. Carries the token counts, model id, temperature, and finish reason.
- **`execute_tool`** — one tool call. Span name `execute_tool {tool_name}`. Carries the tool name and, optionally, sanitized arguments and results.

The parent-child links are what make the trace *navigable*: collapse a worker's `invoke_agent` span and you hide its entire subtree; the orchestrator's view stays clean. This is the observability mirror of the sub-agent isolation you built in orchestration — the context economics and the trace economics are the same shape.

## The required attributes

Attributes are the structured payload on each span. The GenAI conventions name them so backends can aggregate across systems. The ones you will set on nearly every run:

| Attribute key | Span type | Example | Why it matters |
|---|---|---|---|
| `gen_ai.operation.name` | all | `chat`, `execute_tool`, `invoke_agent` | Lets the backend classify the span without parsing the name. |
| `gen_ai.system` | all | `openai`, `anthropic` | Provider, for cross-provider cost and behavior comparison. |
| `gen_ai.request.model` | chat | `gpt-4o-2024-08-06` | The *exact* model — the axis you compare across deploys. |
| `gen_ai.usage.input_tokens` | chat | `1843` | Half of your cost and context-budget signal. |
| `gen_ai.usage.output_tokens` | chat | `412` | The other half. |
| `gen_ai.agent.name` | invoke_agent | `research-worker` | Which agent role, for per-role aggregation. |
| `gen_ai.tool.name` | execute_tool | `web_search` | Which tool, for tool-level success/latency. |
| `gen_ai.response.finish_reasons` | chat | `["stop"]` | `length` or `tool_calls` here explains truncation and loops. |

Prompts and completions are **events** on the span (`gen_ai.user.message`, `gen_ai.choice`, `gen_ai.tool.message`), not attributes — they can be large, and the convention lets you capture or drop them with a single content-capture flag. That flag is a governance decision, not a debugging convenience: prompts and completions routinely contain user PII, and you will revisit it in [mod-309](../mod-309-governance-compliance-domain/README.md).

## Wiring the SDK

A minimal, real OTel Python setup: a tracer provider, an OTLP exporter to whatever backend speaks OTLP, and a batch processor so exporting never blocks the agent loop.

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.resources import Resource
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

resource = Resource.create({
    "service.name": "research-agent",
    "service.version": "2026.06.0",
    "deployment.environment": "production",
})
provider = TracerProvider(resource=resource)
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="https://otel.example.com/v1/traces"))
)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("agent.runtime")
```

The `Resource` is set **once per process** and stamped on every span — `service.version` and `deployment.environment` are what let you slice "quality before vs. after the 2026.06 deploy" in Chapter 3. Get them right here and you never have to backfill them.

## Instrumenting the spans by hand

Auto-instrumentation libraries exist (OpenLLMetry, OpenInference, vendor SDKs), and you will reach for them in production. But you instrument the first one by hand, because the manual version is the only one that shows you exactly which attributes you control.

```python
from opentelemetry.trace import SpanKind

GEN_AI = "gen_ai"

async def run_worker(role: str, instruction: str) -> WorkerResult:
    with tracer.start_as_current_span(
        f"invoke_agent {role}",
        kind=SpanKind.INTERNAL,
        attributes={
            f"{GEN_AI}.operation.name": "invoke_agent",
            f"{GEN_AI}.agent.name": role,
        },
    ) as span:
        result = await agent_loop(role, instruction)
        span.set_attribute(f"{GEN_AI}.usage.input_tokens", result.input_tokens)
        span.set_attribute(f"{GEN_AI}.usage.output_tokens", result.output_tokens)
        return result


async def call_model(messages: list[dict], model: str) -> ModelResponse:
    with tracer.start_as_current_span(
        f"chat {model}",
        attributes={
            f"{GEN_AI}.operation.name": "chat",
            f"{GEN_AI}.system": "openai",
            f"{GEN_AI}.request.model": model,
        },
    ) as span:
        resp = await client.chat(messages=messages, model=model)
        span.set_attribute(f"{GEN_AI}.usage.input_tokens", resp.usage.input_tokens)
        span.set_attribute(f"{GEN_AI}.usage.output_tokens", resp.usage.output_tokens)
        span.set_attribute(f"{GEN_AI}.response.finish_reasons", resp.finish_reasons)
        return resp
```

Because `run_worker` opens its span as the *current* span before calling `agent_loop`, every `chat` and `execute_tool` span created inside the loop attaches to it automatically through OTel's context propagation. You don't pass span objects around — the context carries the parent. That automatic nesting is the whole point: the trace tree builds itself from the call tree.

## Recording failures so they're visible

A worker that throws must leave a span that *says so*, or your trace lies by omission. Set the status and record the exception:

```python
from opentelemetry.trace import Status, StatusCode

try:
    result = await agent_loop(role, instruction)
except Exception as exc:
    span.set_status(Status(StatusCode.ERROR, str(exc)))
    span.record_exception(exc)
    raise
```

A trace where one worker's span is red and the rest are green is worth more than any log line: it shows the failure *in the context of the run that contained it*. This is the same partial-failure visibility your orchestrator needs — now it's queryable.

## Key takeaways

- The GenAI semantic conventions give you **vendor-neutral** span names (`invoke_agent`, `chat`, `execute_tool`) and attribute keys (`gen_ai.*`); instrument against them and the backend becomes swappable.
- The span **hierarchy mirrors the agent topology** — orchestrator `invoke_agent` as root, worker `invoke_agent` spans as children, `chat` and `execute_tool` as leaves — and builds itself from context propagation.
- Set `gen_ai.usage.*` tokens, `gen_ai.request.model`, and the `Resource` (`service.version`, `deployment.environment`) on every run; they are the axes you slice on later.
- The conventions are **experimental** — pin the version, and treat prompt/completion content capture as a governance flag, not a debug toggle.
