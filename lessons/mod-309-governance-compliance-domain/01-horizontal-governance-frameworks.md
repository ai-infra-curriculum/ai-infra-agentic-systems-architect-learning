# Chapter 1 — Horizontal AI Governance Frameworks

Most teams meet governance backwards. A sales deal lands in a regulated sector, legal forwards a statute, and engineers scramble to retrofit controls onto a platform that was never designed to be governed. The result is a pile of sector-specific patches with no common structure — and the next regulated deal starts the scramble over again.

The architect's move is to invert this. **Governance is a horizontal discipline before it is a sectoral one.** There exists a layer of AI governance that applies to *any* AI system — a customer-service agent, a coding agent, a medical-intake agent, a fraud-detection agent — regardless of which statute happens to apply on top. Three frameworks define that horizontal layer: **ISO/IEC 42001** (a management system for AI), the **NIST AI Risk Management Framework** (a risk-management process), and the **EU AI Act** (a risk-tiered regulatory regime). This chapter is about applying those three to an *agentic* architecture, before any sector enters the picture. Get this layer right and the sector-specific work in later chapters becomes a matter of *parameterizing* a governed platform, not rebuilding an ungoverned one.

> **Why "horizontal" first.** If you lead with HIPAA or FERPA, you bake one sector's assumptions into the bones of the system and pay to tear them out for the next sector. If you lead with the horizontal frameworks, you build a platform that is *governable in general*, then set the dials per regime. The horizontal layer is the reusable substrate; the statute is the configuration.

## Three frameworks, three jobs

The three frameworks are not competitors and not interchangeable. They answer different questions, and a mature architecture uses all three.

| Framework | Type | The question it answers | What it gives the architect |
| --- | --- | --- | --- |
| **ISO/IEC 42001** | Management-system standard (certifiable) | *How do we run AI responsibly as an organization, repeatably?* | An AI Management System (AIMS): policies, roles, an Annex A control set, continual improvement. |
| **NIST AI RMF** | Voluntary risk-management framework | *How do we identify, measure, and manage the risk of a given AI system?* | Four functions — Govern, Map, Measure, Manage — and a process to run them. |
| **EU AI Act** | Binding regulation (risk-tiered) | *Is this system even permitted, and what obligations attach to its risk tier?* | A risk classification (unacceptable / high / limited / minimal) and per-tier obligations. |

Read together: the **EU AI Act** tells you *what tier you are in and therefore what you must do*; **NIST AI RMF** gives you a *process* to actually find and treat the risks; **ISO/IEC 42001** gives you the *organizational machinery* to do all of that consistently and prove it. An architect uses the Act (or a sectoral equivalent) to set obligations, RMF to drive the risk work, and 42001 to make it a system rather than a one-off heroic effort.

## ISO/IEC 42001: AI as a management system

ISO/IEC 42001 is the first international standard for an **AI Management System (AIMS)**. If you have seen ISO 27001 (information security) or ISO 9001 (quality), the shape is familiar: it is a management-system standard, which means it specifies *how an organization governs a thing* — not the thing's technical internals. It is **certifiable**, which is its strategic value: you can be audited against it and hold a certificate that a customer's procurement team recognizes.

The standard runs on the **Plan-Do-Check-Act** cycle and carries an **Annex A** catalog of controls — the AI-specific analog to ISO 27001's Annex A. For an architect, the load-bearing parts are:

- **Context and roles.** Who owns AI risk; what the AIMS scope is; which stakeholders (including affected end-users) the system touches.
- **AI policy and objectives.** The organization's stated position on responsible AI, made concrete enough to design against.
- **Annex A controls.** A structured set covering AI system impact assessment, data for AI systems, lifecycle management, transparency to users, and use of AI by third parties — each of which an architect must be able to *point at a component for*.
- **Operation and continual improvement.** Run it, monitor it, find nonconformities, fix them, repeat.

The architect's deliverable against 42001 is a **clause-to-component map**: for each relevant Annex A control, the place in the architecture that satisfies it. A control like "AI system impact assessment" resolves to a documented pre-deployment review gate; "data for AI systems" resolves to a data-provenance and quality boundary; "use of AI by third parties" resolves to the extension-governance lifecycle of [Chapter 4](04-extension-and-tool-governance.md). A control with no component is an open finding.

## NIST AI RMF: a process, in four functions

Where 42001 is the management system, the **NIST AI Risk Management Framework** is the *process you run inside it*. It is voluntary, U.S.-originated, and widely adopted as the common vocabulary for AI risk. It organizes the work into four functions:

```text
   ┌──────────────────────────────────────────────────────────────┐
   │  GOVERN  (cross-cutting; wraps the other three)               │
   │  culture, roles, accountability, policies, risk tolerance     │
   │                                                               │
   │   ┌──────────┐     ┌───────────┐     ┌────────────┐           │
   │   │   MAP    │ ──▶ │  MEASURE  │ ──▶ │   MANAGE   │           │
   │   │ context, │     │ analyze,  │     │ prioritize,│           │
   │   │ risks,   │     │ assess,   │     │ treat,     │           │
   │   │ impacts  │     │ track     │     │ respond    │           │
   │   └──────────┘     └───────────┘     └────────────┘           │
   └──────────────────────────────────────────────────────────────┘
```

- **Govern** is cross-cutting: it establishes the culture, roles, accountability, and risk tolerance that the other three operate within. For an agent fleet, this is *who is accountable for the fleet's behavior* and *what risk we are willing to carry*.
- **Map** establishes context and enumerates risks and impacts — for an agent: what it can do, whom it affects, what could go wrong (tool misuse, data leakage, harmful output, autonomous action beyond intent).
- **Measure** analyzes and tracks those risks with concrete methods — evaluation harnesses ([mod-304](../mod-304-evaluation-harnesses/README.md)), red-teaming, and the observability metrics of [mod-305](../mod-305-observability-tracing/README.md).
- **Manage** prioritizes and treats them — guardrails ([mod-306](../mod-306-guardrails-safety-security/README.md)), HITL gates ([mod-308](../mod-308-deployment-durable-execution/README.md)), tool scoping, and response plans.

The companion **Generative AI Profile** maps these functions onto the specific risks of generative and agentic systems (confabulation, prompt injection, data leakage, autonomy). The architect's deliverable against RMF is to show, for the system's top risks, *which component does the Measuring and which does the Managing* — RMF is the framework that makes "we handle that with guardrails" precise about *which guardrail, measured how*.

## EU AI Act: risk tiers set the obligation

The **EU AI Act** is binding law, and its central architectural idea is the **risk tier**. A system's obligations are a function of which tier it falls in — so the first governance question for any agent is *what tier is this?*

```text
   ┌─────────────────────────────────────────────────────────────┐
   │ UNACCEPTABLE  — prohibited outright (e.g. social scoring,     │
   │                 manipulative or exploitative systems)         │
   ├─────────────────────────────────────────────────────────────┤
   │ HIGH RISK     — permitted with heavy obligations: risk mgmt,  │
   │                 data governance, logging, human oversight,    │
   │                 accuracy/robustness, conformity assessment    │
   ├─────────────────────────────────────────────────────────────┤
   │ LIMITED RISK  — transparency duties (e.g. tell users they are │
   │                 interacting with AI; label AI-generated content)│
   ├─────────────────────────────────────────────────────────────┤
   │ MINIMAL RISK  — no specific obligations                       │
   └─────────────────────────────────────────────────────────────┘
```

The tiers are domain-spanning by design: an agent that screens job applicants, one that assists with medical triage, and one that scores creditworthiness can all land in **high-risk** for entirely different sectoral reasons, yet inherit the *same* structural obligations — a risk-management system, data governance, automatic logging, human oversight, and accuracy/robustness targets. That is the horizontal power of the tier model: it gives the architect a **common obligation set** that later sector-specific rules refine rather than replace.

The Act also imposes obligations on **general-purpose AI / foundation models** and adds **transparency** duties at the limited tier (disclosing AI interaction, labeling synthetic content) that apply to a huge range of agents regardless of sector. For an architect, the Act's logic — *classify, then attach obligations to the class* — is exactly the mental model you will reuse in [Chapter 3](03-multi-regime-mapping.md) when one system meets several regimes at once.

## Applying the frame to an agentic architecture

Agentic systems stress these frameworks in ways a classifier or a chatbot does not, and the architect must name where:

- **Autonomy and action.** An agent does not just predict — it *acts* (calls tools, writes to systems, sends messages). RMF's Map must enumerate the *action* surface, not just the output surface, and the Act's "human oversight" obligation becomes the durable HITL gates of [mod-308](../mod-308-deployment-durable-execution/README.md).
- **Tool and extension surface.** Every tool an agent can call is an expansion of what it can do — and therefore of what must be governed. This is why 42001's "use of AI by third parties" and the extension lifecycle of [Chapter 4](04-extension-and-tool-governance.md) are first-class here, not an afterthought.
- **Non-determinism and confabulation.** The model's output is non-deterministic; RMF's Measure must include evaluation and the GenAI Profile's confabulation risk, and Manage must include output guardrails ([mod-306](../mod-306-guardrails-safety-security/README.md)).
- **Multi-agent emergence.** A fleet of cooperating agents has emergent behavior no single agent's review captures. Govern must assign accountability for the *fleet*, which is the subject of [Chapter 5](05-fleet-governance-and-accountability-specs.md).

The unifying deliverable across all three frameworks is the same artifact in different clothes: a **mapping from a governance requirement to a place in the architecture that satisfies it.** ISO 42001 calls them controls; RMF calls them Manage actions; the Act calls them obligations. The architect's job is identical for all three — make each one resolve to a component, a boundary, a log, a gate, or an owner.

## Worked judgment

*"An internal coding agent that edits files on a sandbox branch, used only by engineers."* — Low autonomy over the outside world, reversible, no affected third parties, no regulated data. Under the Act it is **minimal/limited risk**; under RMF the top risks are code-quality and supply-chain, Managed by code review. A full 42001 AIMS is appropriate at the *organization* level but the system needs a light touch. **Do not over-govern it** — gating every file edit is the failure mode.

*"An agent that screens loan applications and recommends approve/deny."* — High autonomy over consequential outcomes, affects individuals, decision is contestable. Under the Act this is squarely **high-risk** (it inherits risk-management, data-governance, logging, human-oversight, and accuracy obligations) *before* any finance-specific rule (GLBA, fair-lending) is even considered. RMF's Map flags discrimination and explainability risk; Measure adds fairness evaluation; Manage adds a human-decision gate and an audit trail. The horizontal frame already mandates most of the architecture; the sector adds deltas, not foundations.

## Key takeaways

- Govern **horizontally first**: ISO/IEC 42001 (an AI management system), NIST AI RMF (a risk process), and the EU AI Act (risk-tiered obligations) apply to any AI system in any sector. Lead with them so the platform is *governable in general* before any statute is added.
- The three frameworks have **distinct jobs** — the Act sets the obligation *tier*, RMF gives the *process* (Govern / Map / Measure / Manage), 42001 gives the *management system* to run it repeatably and prove it.
- Agentic systems stress the frameworks at the **action, tool, non-determinism, and multi-agent** surfaces; an architect must map governance onto *what the agent does*, not just what it outputs.
- Every framework requirement must resolve to a **component, boundary, log, gate, or owner**. A control with no place in the architecture is an open finding — this clause-to-component discipline is the throughline of the entire module.
