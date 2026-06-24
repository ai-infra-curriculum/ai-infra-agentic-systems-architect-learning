# Resources for mod-304-evaluation-harnesses

Primary references for building evaluation harnesses for agentic systems. Verify against current docs — eval tooling and standards move fast.

## Trajectory, final-state, and tool-call evaluation

- **OpenAI Evals** ([github.com/openai/evals](https://github.com/openai/evals)) — open framework for building and running evals; useful for the registry/dataset/scorer structure even if you don't use it directly.
- **RAGAS** ([docs.ragas.io](https://docs.ragas.io)) — reference-free and reference-based metrics for retrieval-grounded systems (faithfulness, answer relevancy, context precision/recall). Directly relevant to the citation-faithfulness axis.
- **LangChain — Agent / trajectory evaluation** ([docs.langchain.com/oss/python/langsmith/evaluation](https://docs.langchain.com/oss/python/langsmith/evaluation)) — trajectory and tool-call evaluators, including required-tool and exact/superset trajectory matching.

## LLM-as-judge

- **Zheng et al. — Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena** ([arxiv.org/abs/2306.05685](https://arxiv.org/abs/2306.05685)) — the canonical study of LLM-as-judge: agreement with humans, plus position, verbosity, and self-enhancement bias. Read before authoring a judge you intend to gate on.
- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the evaluator-optimizer pattern and the discipline of measuring agents rather than eyeballing them.
- **Anthropic — A statistical approach to model evaluations** ([anthropic.com/research/statistical-approach-to-model-evals](https://www.anthropic.com/research/statistical-approach-to-model-evals)) — how to report eval results with error bars; informs the "k runs per case, gate on the distribution" discipline.

## Observability and traces (the raw material trajectory eval scores)

- **OpenTelemetry — GenAI semantic conventions** ([opentelemetry.io/docs/specs/semconv/gen-ai](https://opentelemetry.io/docs/specs/semconv/gen-ai/)) — the emerging standard for span/event attributes on LLM and agent calls (tool calls, tokens, model). Emit traces to this shape so your harness and tooling stay portable.
- **LangSmith** ([docs.smith.langchain.com](https://docs.smith.langchain.com)) — tracing, datasets, and evaluation runs (offline and online), including LLM-as-judge evaluators and dataset versioning.
- **Langfuse** ([langfuse.com/docs](https://langfuse.com/docs)) — open-source LLM observability with datasets, scores, and evals; self-hostable. Good for the online/continuous-scoring half of the strategy.
- **Arize Phoenix** ([docs.arize.com/phoenix](https://docs.arize.com/phoenix)) — open-source, OpenTelemetry-native tracing and evaluation; ships LLM-as-judge "evals" templates and integrates with the GenAI semconv above.

## Strategy and gating

- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — real-world account of evaluating an agentic system, including starting small and curating eval cases from real failures.
- **Hugging Face — Evaluate** ([huggingface.co/docs/evaluate](https://huggingface.co/docs/evaluate)) — a library of metrics and the comparison/measurement vocabulary useful when assembling an offline regression suite.

> You'll build the scorers and the gate by hand in this module. When you adopt a platform (LangSmith, Langfuse, Phoenix), you'll recognize exactly which of these pieces — datasets, trajectory evaluators, LLM-judge templates, online scoring — it's packaging, and you'll trust and debug it faster for having built it yourself. See also [mod-305: Observability & Tracing](../mod-305-observability-tracing/README.md).
