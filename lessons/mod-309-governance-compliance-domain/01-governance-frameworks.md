# Chapter 1 — Governance Frameworks for Agentic Systems

A governance framework is not a binder. For an architect, it is a **map from obligations to components**: each clause of a standard should resolve to something you can point at in your system — a control, a boundary, a log, an approval gate, or a named owner. This chapter takes the two frameworks that matter most for an agentic platform — **ISO/IEC 42001** and the **NIST AI Risk Management Framework** — and shows how to apply them to a fleet of autonomous, tool-using agents rather than to a single static model.

## Why an agentic system raises the stakes

A predictive model scores an input and returns a number. An agentic system *plans, calls tools, takes actions in external systems, and runs in a loop* — often unattended. That changes the governance surface in four ways:

- **Autonomy.** The system decides *what to do next*, not just *what to predict*. Governance must constrain the action space, not only the output.
- **Tool reach.** An agent with a `send_email`, `update_record`, or `issue_refund` tool can change the world. Every tool is an authority that must be scoped, logged, and revocable.
- **Emergent behavior.** Multi-step plans and multi-agent handoffs produce behavior no single prompt specified. You govern the *system*, not just each call.
- **Non-determinism.** The same input can yield different actions. Reproducibility, and therefore auditability, must be engineered in (trace IDs, pinned model/prompt versions, recorded tool I/O) — it is not free.

The frameworks below were written to be technology-neutral. Your job is to make them agentic-specific.

## ISO/IEC 42001 — an AI management system

ISO/IEC 42001:2023 is the first certifiable **AI management system (AIMS)** standard. It is structured like ISO/IEC 27001 (the information-security management system standard) and shares the same Annex SL high-level structure, so if your organization already runs a certified ISMS, the AIMS bolts onto it rather than replacing it. It is a **management-system** standard: it governs how you *run* the practice of building and operating AI, with a continual **Plan-Do-Check-Act** improvement cycle — it is not a checklist of model-level technical tests.

The numbered management-system clauses (the auditable "shall" requirements) are:

| Clause | Title | What it means for an agent platform |
| --- | --- | --- |
| 4 | Context of the organization | Define the AIMS scope: which agents, fleets, and use cases are in scope; identify interested parties (users, minors, regulators). |
| 5 | Leadership | An accountable owner for the AIMS exists; an AI policy is published and resourced. |
| 6 | Planning | AI risk assessment and **AI risk treatment**; an **AI system impact assessment** for affected individuals and society; measurable objectives. |
| 7 | Support | Resources, competence, awareness, and **documented information** (your records and evidence). |
| 8 | Operation | Operational planning and control; run the impact assessment and risk treatment in practice. |
| 9 | Performance evaluation | Monitoring, measurement, internal audit, and management review. |
| 10 | Improvement | Nonconformity, corrective action, and continual improvement. |

Annex A is a **reference set of controls and objectives** you select from based on your risk assessment (the same shape as ISO/IEC 27001 Annex A). Its control areas include policies for AI, internal organization and roles, resources for AI systems (data, tooling, compute, human resources), AI system impact assessment, the AI system life cycle, data for AI systems, information for interested parties, use of AI systems, and third-party and customer relationships. Annex B gives implementation guidance for those controls; Annex C lists potential AI-related objectives and risk sources. **You do not implement every Annex A control blindly** — you justify inclusion or exclusion against your risk assessment in a statement-of-applicability style artifact, exactly as in an ISMS.

> **Architect's move.** Treat clause 6 (planning / risk treatment) and the Annex A life-cycle and third-party controls as your hooks. The risk assessment tells you *which* agent capabilities are high-risk; the life-cycle controls tell you *what to do at each stage*; the third-party controls are where **plugin governance** (Chapter 3) lives.

## NIST AI Risk Management Framework

The NIST AI RMF (AI 100-1, version 1.0, released January 2023) is **voluntary, non-certifiable, and U.S.-origin**, but it is the most widely adopted operational vocabulary for AI risk. It is organized around four **functions**, each broken into categories and subcategories:

```text
GOVERN  — the culture, policies, accountability, and oversight that
          run across the other three functions (it is not a phase;
          it is always on)
MAP     — establish context and frame risk: what is the system,
          who does it affect, what could go wrong
MEASURE — analyze, assess, benchmark, and monitor the mapped risks
          using quantitative and qualitative methods
MANAGE  — allocate resources to, respond to, and recover from the
          measured risks; prioritize and act
```

NIST also publishes a **Generative AI Profile (NIST AI 600-1, July 2024)** that enumerates risks specific to generative and, by extension, agentic systems — confabulation (hallucination), data leakage, harmful or biased content, and the amplification of harm through automation. Use the GenAI Profile as the catalog you draw your agent-specific risks from when you run the MAP function.

ISO/IEC 42001 and the NIST AI RMF are complementary, not competing. A common pattern: **certify against ISO/IEC 42001** for the management system and external assurance, and **operate day-to-day with the NIST AI RMF functions** because they give crisper, more testable language for risk. NIST publishes a crosswalk showing how RMF subcategories line up with ISO/IEC 42001 clauses.

## Mapping a framework onto a real architecture

The deliverable that makes governance real is a **controls-mapping table**: framework requirement → architectural realization → evidence. Without the third column you have intentions, not controls. Here is the shape, with agent-specific rows:

| Framework requirement | Architectural realization | Evidence (what an auditor inspects) |
| --- | --- | --- |
| 42001 Cl. 6 — AI risk treatment for high-risk capabilities | Tool-permission tiers; high-impact tools (`issue_refund`, `delete_record`) require a human approval gate | Risk register; approval-gate config; HITL approval logs |
| 42001 Annex A — AI system life cycle | Gated promotion pipeline (eval → staging → prod) with release sign-off | Pipeline definition; eval-suite results per release; sign-off records |
| 42001 Annex A — third-party / supplier controls | Plugin registry with vetting gate and signed manifests (Chapter 3) | Plugin approval records; signature verification logs |
| NIST MAP — context & impact | AI system impact assessment per use case, listing affected individuals (incl. minors) | Completed impact assessments, versioned and dated |
| NIST MEASURE — monitor mapped risks | Eval harness + observability traces with per-action attributes (mod-304, mod-305) | Dashboards; trace store; eval-regression history |
| NIST MANAGE — respond & recover | Incident runbook; kill switch that revokes tool grants fleet-wide | Runbook; documented kill-switch drill; incident records |
| NIST GOVERN — accountability | RACI naming an accountable owner per agent and per fleet | Org chart / RACI; decision records (ADRs) |

Two rules keep this honest:

- **Every requirement maps to a component *and* an evidence source.** "We are committed to fairness" is not a control. "Output bias evaluated each release by suite X; results stored in Y; owner Z" is.
- **Map exclusions explicitly.** If you exclude an Annex A control, write *why* (e.g., "no automated decisions about individuals are made by this fleet"). Auditors care as much about justified exclusions as inclusions.

## What you are not doing

You are not turning your team into a compliance department, and you are not implementing every control a standard mentions. You are selecting the controls your risk assessment justifies, realizing each as something inspectable in the architecture, and recording the evidence trail. Over-governance — gates on low-risk read-only agents, sign-offs that no one reads — is its own failure mode: it trains the organization to route around governance. Scale the control to the risk.

## Key takeaways

- Governance frameworks are maps from obligations to components; a control you cannot point at in the architecture does not exist.
- **ISO/IEC 42001** is a certifiable *management-system* standard (clauses 4–10, Annex A reference controls selected by risk); **NIST AI RMF** is a voluntary *risk* framework (GOVERN/MAP/MEASURE/MANAGE) with a GenAI Profile for agent-specific risks. Use them together.
- Agentic systems raise the stakes through autonomy, tool reach, emergent behavior, and non-determinism — govern the action space and the system, not just the model output.
- The load-bearing artifact is a **controls-mapping table** with a third column for evidence; map exclusions as deliberately as inclusions, and scale control rigor to risk.
