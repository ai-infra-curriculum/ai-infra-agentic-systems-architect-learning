# Resources for mod-305-observability-tracing

Primary references for agent observability and tracing. The OTel GenAI conventions are **experimental** and the platform market moves monthly — verify everything against current docs and pin versions.

## OpenTelemetry GenAI semantic conventions

- **OTel GenAI semantic conventions** ([opentelemetry.io/docs/specs/semconv/gen-ai](https://opentelemetry.io/docs/specs/semconv/gen-ai/)) — the registry of `gen_ai.*` span names, attributes, and events. Start here; this is the contract you instrument against.
- **GenAI agent and tool spans** ([opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans/)) — the `invoke_agent`, `chat`, and `execute_tool` operation definitions and their required/recommended attributes.
- **OpenTelemetry Python SDK** ([opentelemetry.io/docs/languages/python](https://opentelemetry.io/docs/languages/python/)) — tracer provider, span processors, OTLP exporter, and context propagation used in the chapter examples.
- **OpenLLMetry** ([github.com/traceloop/openllmetry](https://github.com/traceloop/openllmetry)) — open-source auto-instrumentation emitting OTel GenAI spans; the library version of the manual instrumentation in Chapter 1.

## Concepts: agent observability vs. APM

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the agent patterns you're tracing; the failure modes here are what your quality signals must catch.
- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — real-world observability and failure analysis of a production orchestrator-worker system.
- **OpenTelemetry — observability primer** ([opentelemetry.io/docs/concepts/observability-primer](https://opentelemetry.io/docs/concepts/observability-primer/)) — traces, spans, and context propagation if you need the foundations before Chapter 1.

## Platforms (for the Chapter 4 evaluation)

- **Arize Phoenix** ([docs.arize.com/phoenix](https://docs.arize.com/phoenix)) — open-source, OpenInference/OTel-native tracing with strong eval, drift, and embedding/cluster analysis. Self-hosts trivially; good local dev-loop companion.
- **Langfuse** ([langfuse.com/docs](https://langfuse.com/docs)) — open-source (MIT core), self-hostable, OTel-native tracing with evals, datasets, and prompt management. The data-ownership default.
- **LangSmith** ([docs.smith.langchain.com](https://docs.smith.langchain.com)) — the most polished managed eval+observability product, with deep dataset/experiment and prompt tooling; OTel ingestion alongside its native SDK.
- **OpenInference** ([github.com/Arize-ai/openinference](https://github.com/Arize-ai/openinference)) — the OTel-compatible convention behind Phoenix; useful when comparing convention coverage across backends.

> You designed the span schema, the signal taxonomy, and the platform-evaluation matrix by hand in this module. Every platform above is packaging some subset of those decisions — instrument against the open OTel GenAI conventions and the backend stays a swappable component. See [mod-304: Evaluation Harnesses](../mod-304-evaluation-harnesses/README.md) for the eval rubrics you attach to traces.
