# mod-309-governance-compliance-domain: Governance, Compliance & Regulated-Domain Architecture

**Estimated effort:** 15 hours

By the time an agentic platform reaches production, the hard questions are no longer "can the agent do it?" — they are "are we *allowed* to ship it, can we *prove* we governed it, and who is *accountable* when it goes wrong?" Those are architecture problems, not afterthoughts. A fleet of autonomous agents that takes actions, calls tools, ingests sensitive data, and produces output that affects real people is a governed system whether or not anyone wrote it down. This module is where you, as the **architect**, make the governance explicit — and you do it the right way around: **horizontal AI governance first** (a discipline that applies to any AI system in any sector), then the *generic* method for turning regulatory constraints into architecture, and only then the sector-specific deltas.

> **Governance is design, not paperwork.** A control you cannot point at in the architecture is a control you do not have. Every framework clause in this module resolves to a component, a boundary, a log, an approval gate, or a named owner — something a reviewer can inspect. If a requirement does not change the system diagram, you have not finished translating it.

The thread running through every chapter: **regulated domains are a parameter, not a special case.** A healthcare deployment under HIPAA, a finance deployment under GLBA/SOX/PCI, a public-sector deployment, and an edtech deployment under FERPA/COPPA are not four unrelated problems — they are one governed architecture with four different settings of the same dials (privacy, residency, auditability, accountability, retention). Learn the dials once and you can re-tune the same platform for any regime, instead of rebuilding it per statute.

## 🎯 Purpose

This module pitches up from *building* compliant features to *architecting* a governable platform. You will:

- Adopt a **horizontal AI governance frame** (ISO/IEC 42001, NIST AI RMF, EU AI Act risk tiers) and map its clauses and functions onto a concrete agentic architecture — before any sector enters the picture.
- Learn the **statute-independent translation method**: how to turn any regulatory constraint — privacy, residency, auditability, accountability, retention — into a component, boundary, log, or owner.
- Map **one architecture against multiple regimes at once** and reason about where controls *converge* (build once) versus *diverge* (parameterize), treating healthcare, finance, public sector, and edtech as equal peers.
- Govern the **extension surface** — third-party and first-party agent tools and plugins — through an evaluate → version → approve → monitor lifecycle, so an extensible platform stays a governed one.
- Produce the **governance, accountability, and risk-control specifications** (lineage, audit trails, human-accountability points) a fleet operator runs against and an auditor reviews.

## Lecture chapters

| # | Chapter | Focus |
| --- | --- | --- |
| 1 | [Horizontal AI Governance Frameworks](01-horizontal-governance-frameworks.md) | ISO/IEC 42001 as an AI management system, the NIST AI RMF functions, and EU AI Act risk tiers — applied to an agentic architecture, sector-neutral. |
| 2 | [Regulated Domain as Architecture](02-regulated-domain-as-architecture.md) | The generic translation method: turning privacy, residency, auditability, accountability, and retention constraints into architecture, independent of any single statute. |
| 3 | [Multi-Regime Mapping](03-multi-regime-mapping.md) | One architecture against multiple regimes; convergence vs. divergence; healthcare, finance, public sector, and edtech as balanced peers. |
| 4 | [Extension and Tool Governance](04-extension-and-tool-governance.md) | Governing agent extensions, plugins, and tools through evaluate → version → approve → monitor, for both third-party and internal components. |
| 5 | [Fleet Governance and Accountability Specs](05-fleet-governance-and-accountability-specs.md) | Lineage, audit trails, and human-accountability points — the specifications that govern an agent fleet at scale. |

## Exercises

Hands-on architecture and specification practice — control maps, comparison tables, lifecycle designs, and specs, not framework wiring. Reference solutions live in the paired [solutions repo](https://github.com/ai-engineering-curriculum/agentic-systems-architect-solutions).

| # | Exercise | Deliverable |
| --- | --- | --- |
| 1 | [Horizontal framework controls mapping](exercises/exercise-01-horizontal-framework-controls-mapping.md) | Map an agentic system onto ISO/IEC 42001 + NIST AI RMF controls, sector-neutral. |
| 2 | [Multi-regime regulated-domain architecture](exercises/exercise-02-multi-regime-regulated-domain-architecture.md) | One reference agent; control deltas across healthcare, finance, public sector, and edtech as equal peers, plus a convergence/divergence table. |
| 3 | [Extension and tool governance lifecycle](exercises/exercise-03-extension-tool-governance-lifecycle.md) | Design evaluate → version → approve → monitor for third-party and internal agent tools/plugins. |
| 4 | [Governance and accountability spec](exercises/exercise-04-governance-accountability-spec.md) | Author the lineage, audit-trail, and human-accountability spec for an agent fleet. |
| 5 | [Data-handling and residency design](exercises/exercise-05-data-handling-residency-design.md) | Design privacy/residency/retention controls parameterized by regime, not hardcoded to one. |

## Assessment

- [Quiz 1 — Governance, Compliance & Regulated-Domain Architecture](quizzes/quiz-01-governance-compliance-domain.md) — covers all five chapters.

## Prerequisites

This is the **architect** rung. It assumes you can already build agents, tools, and RAG pipelines — those skills are owned by the lower [Senior Agentic AI Engineer](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (L30) track and are linked, not re-taught. Here you decide *whether, where, and how* governance belongs in a production architecture.

Within this track:

- [mod-306: Guardrails, Safety & Security](../mod-306-guardrails-safety-security/README.md) — the technical controls (input/output filtering, tool scoping, sandboxing) that governance clauses reference and require.
- [mod-308: Deployment, Durable Execution & Human-in-the-Loop](../mod-308-deployment-durable-execution/README.md) — the durable HITL approval flows and fleet operation that this module governs and audits.
- [mod-305: Observability & Tracing](../mod-305-observability-tracing/README.md) — the traces and logs that make auditability real rather than aspirational.

## How this module fits

Governance is the connective tissue across the architect track. The guardrails of mod-306 become the *controls* a framework clause points at; the HITL gates of mod-308 become *accountability points* in an audit trail; the traces of mod-305 become the *lineage* an auditor replays. This module gives you the frame that ties them together — and the discipline to express any regulatory requirement, in any sector, as something a reviewer can inspect in the architecture itself.

See [resources.md](resources.md) for the primary standards and regulatory sources. **Cite the primary source, not a blog summary** — in governance work, the clause number is the evidence.
