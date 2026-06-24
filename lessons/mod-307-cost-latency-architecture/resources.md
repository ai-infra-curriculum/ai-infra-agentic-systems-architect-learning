# Resources for mod-307-cost-latency-architecture (Cost & Latency Architecture)

Primary references for cost and latency architecture. **Pricing, cache rates, and batch discounts change** — always pull current numbers from the provider pricing page before committing a design to a budget. The methods in this module are durable; the figures are not.

## Pricing and token economics

- **Anthropic — Pricing** ([anthropic.com/pricing](https://www.anthropic.com/pricing) and the API docs at [docs.anthropic.com/en/docs/about-claude/pricing](https://docs.anthropic.com/en/docs/about-claude/pricing)) — per-model input/output token prices, cache read/write rates, and batch discount. The source of truth for every number you plug into the cost model.
- **Anthropic — Models overview** ([docs.anthropic.com/en/docs/about-claude/models/overview](https://docs.anthropic.com/en/docs/about-claude/models/overview)) — the model tiers (Opus / Sonnet / Haiku), their relative cost and capability, and context-window sizes. The basis for the routing-tier decision in Chapter 2.
- **Anthropic — Token counting** ([docs.anthropic.com/en/docs/build-with-claude/token-counting](https://docs.anthropic.com/en/docs/build-with-claude/token-counting)) — count tokens before you send them, so per-call estimates in the cost model are grounded, not guessed.

## The architectural levers

- **Anthropic — Prompt caching** ([docs.anthropic.com/en/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)) — `cache_control`, the ephemeral TTL, write/read pricing, cache-first prompt ordering, and the `cache_creation_input_tokens` / `cache_read_input_tokens` usage fields. The anchor text for Lever 1.
- **Anthropic — Message Batches API** ([docs.anthropic.com/en/docs/build-with-claude/batch-processing](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing)) — the asynchronous ~50%-off lane, the processing window, request limits, and result retrieval. The anchor text for Lever 3.
- **Anthropic — Reducing latency** ([docs.anthropic.com/en/docs/build-with-claude/latency](https://docs.anthropic.com/en/docs/build-with-claude/latency)) — model selection, prompt length, streaming, and other latency levers feeding the cost–latency–quality trade in Chapter 4.
- **Anthropic — Streaming Messages** ([docs.anthropic.com/en/docs/build-with-claude/streaming](https://docs.anthropic.com/en/docs/build-with-claude/streaming)) — streaming changes *perceived* latency (time-to-first-token) without changing cost; relevant to the interactive-surface SLA.

## Patterns and real-system economics

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the workflow-vs-agent line and the "start simple, add complexity only when it improves outcomes" discipline that the value threshold (Chapter 1) puts a price on.
- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — the source of the ~15× token-multiplier figure for fan-out systems and a candid account of multi-agent cost economics and failure modes.

## Reliability economics (cross-module)

- **Google SRE — Service Level Objectives** ([sre.google/sre-book/service-level-objectives](https://sre.google/sre-book/service-level-objectives/)) — SLI/SLO/SLA discipline and **why you target percentiles (p95/p99), not the mean** — applied to cost-as-an-SLO in Chapter 3. See also [mod-305: Observability & Tracing](../mod-305-observability-tracing/README.md) for wiring the meter to dashboards and alerts.

> You priced the architecture and wired in the budgets here. When you adopt a gateway, a cost dashboard, or an LLM proxy that advertises caching, routing, and batching, you will recognize exactly which lever it packages — and you will know which precondition to check before you trust its savings claim.
