# Chapter 4 — Evaluating Observability Platforms

You've designed the span schema (Chapter 1), the signal taxonomy (Chapter 2), and the identity model (Chapter 3). Now you choose where the traces *land*. This is a build-vs-buy architecture decision with a multi-year blast radius: the platform shapes how your team debugs, how evals run, what you pay, and how hard it is to leave. The wrong way to make it is a feature-checklist bake-off. The right way is to derive **requirements from your own system**, weight them, and run a real proof-of-concept against your actual traces. This chapter gives you the criteria and an honest read of the three platforms you'll most often shortlist — **LangSmith, Langfuse, and Arize Phoenix** — current as of mid-2026 and worth re-verifying, because this market moves monthly.

## The criteria that actually decide it

A requirements matrix, not a feature list. Each criterion is something a *system* needs, not a checkbox a vendor ticks.

- **OTel-native ingestion.** Does it accept the GenAI-convention OTLP you instrumented in Chapter 1, or does it require *its own* SDK? An OTel-native backend keeps your instrumentation portable; a proprietary-SDK-only backend re-locks you in even though you used the open convention.
- **Self-host vs. managed.** Can you run it in your own VPC? For regulated data (healthcare, finance — [mod-309](../mod-309-governance-compliance-domain/README.md)), traces contain prompts full of PII, and "send them to a third-party SaaS" may be a non-starter.
- **Evaluation integration.** Can it run LLM-judge and code evals *on the captured traces* and join scores back by `trace_id`? This is the bridge to [mod-304](../mod-304-evaluation-harnesses/README.md) — the quality plane of Chapter 2 lives or dies on it. Online (production sampling) and offline (dataset) both matter.
- **Cost model and scale.** Per-trace? Per-seat? Per-ingested-event? Self-host compute? At your trace volume, the pricing axis can swing the annual cost 10x. Model it against your *projected* volume, not today's.
- **Datasets, experiments, and prompt management.** Can you curate failing traces into a regression dataset, version prompts, and diff experiments across `prompt.version`? This is what turns observability into an improvement loop instead of a read-only dashboard.
- **Framework coupling.** Is it tied to one orchestration framework, or framework-agnostic via OTel? A platform that only deeply understands one framework is a bet on that framework.
- **Operational maturity.** RBAC, SSO, data retention controls, alerting, dashboards-as-code. Boring, and the reason migrations happen.

## The three platforms, honestly

A read on the common shortlist. Verify against current docs — versions and licensing change.

**LangSmith** (LangChain). The most polished managed product, with the deepest eval and dataset tooling and excellent prompt management. Framework-agnostic in principle and strong with LangChain/LangGraph in practice. OTel ingestion is supported alongside its native SDK. Primarily SaaS (self-hosted/enterprise tiers exist on higher plans); pricing is seat- and trace-based. **Choose it when** you want the most batteries-included eval+observability product and a managed SaaS is acceptable, especially in a LangChain shop.

**Langfuse**. **Open-source (MIT core)** and genuinely self-hostable — the strongest answer when data residency or "no third-party SaaS" is a hard constraint. OTel-native ingestion, solid tracing, evals, datasets, prompt management, and a generous managed free tier. Slightly less turnkey than LangSmith on the eval-tooling depth, but the self-host story and open licensing make it the default when you must own the data. **Choose it when** self-hosting, open licensing, or cost control at scale outweigh maximum out-of-box polish.

**Arize Phoenix** (Arize AI). **Open-source and built on OpenInference** (an OTel-compatible convention), with a research-grade lean toward **evaluation, drift, and embedding/cluster analysis** — strongest of the three at surfacing the *drift* signal from Chapter 2. Runs locally or self-hosted trivially (great for dev-loop tracing), with Arize's commercial platform for production scale. **Choose it when** evaluation depth and drift/embedding analysis are the priority and you want OTel-native, open tooling — or as a local tracing companion alongside another production backend.

```text
                  OTel-native   Self-host   Eval depth   Drift/embed   License
LangSmith            yes*        ent. tier     ★★★★★        ★★★         commercial
Langfuse             yes         yes (easy)    ★★★★         ★★★         OSS (MIT core)
Arize Phoenix        yes         yes (easy)    ★★★★★        ★★★★★       OSS
   * supported alongside a native SDK; verify convention coverage for your spans
```

This table is a *starting hypothesis*, not a verdict. The weights are yours.

## The build option, and why "buy" usually wins

You *can* point OTLP at your own stack: an OTel Collector into ClickHouse or Postgres, Grafana for dashboards, a homegrown LLM-judge job joining scores by `trace_id`. The tracing half is genuinely tractable — OTel is open and the storage is commodity. The half that sinks build projects is everything in Chapter 2's quality plane: the eval runner, the dataset curation UI, the prompt-diff experiments, the drift analytics. That's a *product*, not a weekend. Build only when you have a hard constraint no vendor meets (extreme scale economics, an air-gapped environment, a data-residency rule with no self-host option) **and** the standing team to own it. For nearly everyone else, the open-source self-host options (Langfuse, Phoenix) capture most of the "own your data" benefit without the product-build cost.

## The decision artifact

Don't decide in a meeting; produce an artifact. A **weighted scoring matrix** — criteria as rows, weights you assigned, platforms as columns, scores from a *real proof-of-concept on your own traces* — is the deliverable. The POC is non-negotiable: ingest a day of real (or realistic) traffic, run one LLM-judge eval, curate one regression dataset, and check that drift-over-deploy query from Chapter 3 actually works. Demos lie; your own traces don't. You build exactly this matrix in [exercise-03](exercises/exercise-03-observability-platform-evaluation.md).

## Key takeaways

- Choose a platform by **requirements derived from your system and weighted**, then a **real proof-of-concept on your own traces** — never a feature-checklist bake-off.
- The deciding criteria are **OTel-native ingestion, self-host vs. managed, eval integration, cost model at your scale, dataset/experiment tooling, framework coupling, and operational maturity.**
- **LangSmith** = most polished managed eval+observability; **Langfuse** = open-source, self-hostable, data-ownership default; **Arize Phoenix** = open, OTel-native, strongest on eval and drift/embedding analysis. Verify current; the market moves monthly.
- **Build** only behind a hard constraint plus a standing team — the tracing half is tractable, but the eval/dataset/drift *product* is what sinks DIY projects; self-hosted OSS captures most of the ownership benefit.
