# Regulated-Domain Agentic Architecture (Learner-Selected Sector)

**Duration**: 20 hours | **Difficulty**: High | **Project ID**: project-302-regulated-domain-agent-architecture

## Overview

Agentic systems become much harder the moment they touch regulated data and high-stakes decisions. This project asks you to produce a reference architecture for an agentic system operating under regulated-domain constraints, then prove your design is principled — not sector-specific accident — by writing a **portability analysis** showing which controls change if the same system were deployed in a different regulated sector.

The point is to internalize the difference between controls that are intrinsic to responsible agentic design (auditability, human-in-the-loop on high-risk output, hallucination containment, toggleable AI features) and controls that are shaped by a specific regulatory regime (data residency, consent, retention, breach notification). A strong submission makes that boundary explicit.

### Pick one sector from a balanced menu

Choose exactly one primary sector. No sector is privileged or assumed; pick the one you can reason about most rigorously:

| Sector | Representative regime(s) | Characteristic constraints |
|--------|--------------------------|-----------------------------|
| Healthcare | HIPAA | PHI handling, minimum-necessary access, audit trails, business-associate boundaries |
| Finance | GLBA, SOX, PCI DSS | Nonpublic personal information, financial-reporting integrity, cardholder-data isolation |
| Public sector | Records, accessibility, and procurement obligations | Records retention, transparency/FOIA-style disclosure, accessibility, authorization-to-operate |
| Education technology | FERPA, COPPA | Education-record privacy, parental consent, age-gating, vendor-data-use limits |

You will design for your chosen sector and then, in the portability analysis, re-derive the controls for **at least one other sector from this menu**. Treat the menu as genuinely balanced: do not default to the sector that seems easiest, and do not assume the curriculum favors any one of them.

## Learning Objectives

By completing this project, you will:

1. Translate a regulatory regime into concrete architectural controls for an agentic system.
2. Distinguish regime-specific controls from domain-agnostic responsible-AI controls.
3. Design data-handling, residency, auditability, and accountability into the architecture from the start.
4. Place human-in-the-loop checkpoints on high-risk agent output and justify the risk threshold.
5. Contain hallucination by grounding output in vetted sources and refusing ungrounded high-risk claims.
6. Reason about cross-regime portability so the architecture is robust rather than over-fitted.

## Project Scenario

### Context

You are the architect for an agentic system that operates inside a regulated environment of your chosen sector. The system performs real work over sensitive records — answering questions, drafting documents, or taking bounded actions — and its output can affect people's care, money, rights, or records. A compliance reviewer and an auditor will both scrutinize the design. Neither will accept "the model is usually right" as a control.

Leadership also wants the option to **turn AI features off** — per tenant, per feature, or globally — without taking the rest of the product down, because some customers, regions, or incidents will require it.

### Your mission

Produce the regulated-domain reference architecture for your chosen sector, then write the portability analysis that shows which controls would change for a second sector on the menu and which controls stay the same regardless.

## Requirements

### Functional requirements (required across all sector choices)

1. **FR-1 — Data-handling and residency controls**: Classify the sensitive data the agent touches, and design handling controls: encryption in transit and at rest, residency/region constraints, minimization (only the data the task needs reaches the model), and retention/deletion policy.
2. **FR-2 — Auditability and accountability**: Every consequential agent action and model decision is logged with who/what/when/why and the inputs that produced it, in a tamper-evident, queryable audit trail. Accountability — which human or role owns each automated decision — is explicit.
3. **FR-3 — Human-in-the-loop on high-risk output**: Define a risk taxonomy for outputs/actions and require human review/approval above a stated threshold; lower-risk output may flow automatically with logging.
4. **FR-4 — Hallucination containment**: High-risk factual output must be grounded in vetted sources with citations; the agent must be able to refuse or escalate when it cannot ground a claim. Define what "vetted source" means for your sector.
5. **FR-5 — Toggleable AI features**: AI capabilities can be disabled at global, tenant, and per-feature granularity, with a defined safe degraded mode (what the product does when AI is off) and an audit record of the toggle.

### Portability analysis (required)

6. **FR-6 — Cross-regime portability analysis**: For at least one other sector on the menu, re-derive FR-1 through FR-5. State, per control, whether it is **unchanged**, **tightened**, **relaxed**, or **replaced**, and why. Conclude with a short statement of which controls are regime-agnostic responsible-AI controls and which are regime-specific.

### Non-functional requirements

1. **Defensibility**: A compliance reviewer can trace each regulatory obligation to a specific control in the architecture (a control-mapping matrix).
2. **Least privilege**: The agent and its tools hold the minimum access required; access is scoped and revocable.
3. **Observability**: Audit and trace data are sufficient to reconstruct any consequential decision after the fact.
4. **Fail-safe**: When grounding, authorization, or a required human approval is unavailable, the system fails closed (refuse/escalate), never open.

### Constraints

- Exactly one primary sector; at least one secondary sector for the portability analysis.
- No real regulated data — use synthetic or clearly fictitious records throughout.
- Do not privilege any sector; the design rationale must hold up for whichever you pick.

## Deliverables

Suggested layout:

```text
project-302-regulated-domain-agent-architecture/
├── README.md                    # This file
├── architecture/                # Reference architecture + diagrams
├── control-mapping/             # Obligation -> control traceability matrix
├── data-handling/               # Data classification, residency, retention
├── hitl/                        # Risk taxonomy + approval flows
├── grounding/                   # Vetted-source + hallucination-containment spec
├── toggles/                     # Toggleable-AI + degraded-mode spec
├── portability/                 # Cross-regime portability analysis
└── adrs/                        # Key decisions (>=6)
```

1. **Reference architecture + diagrams** — Container and sequence views showing where sensitive data flows, where audit logging occurs, where HITL gates sit, and where grounding is enforced.
2. **Control-mapping matrix** — Each regulatory obligation for your sector mapped to a specific architectural control and to the requirement it satisfies.
3. **Data-handling specification** — Data classification, residency/region rules, minimization at the model boundary, and retention/deletion policy.
4. **Risk taxonomy + HITL flows** — The output/action risk tiers, the approval threshold, and the approval/escalation flows with audit hooks.
5. **Grounding + hallucination-containment spec** — Definition of vetted sources, the citation/grounding mechanism, and the refuse-or-escalate behavior for ungrounded high-risk claims.
6. **Toggleable-AI + degraded-mode spec** — Toggle granularity (global/tenant/feature), the safe degraded mode, and the toggle audit record.
7. **Portability analysis** — The cross-regime re-derivation for a second sector with the unchanged/tightened/relaxed/replaced classification and the regime-agnostic vs. regime-specific conclusion.
8. **ADR set (>=6)** — Sector choice, grounding strategy, HITL threshold, residency approach, toggle design, and audit-log design.

## Effort breakdown (20 hours)

| Phase | Focus | Hours | Primary deliverable |
|-------|-------|-------|---------------------|
| 1 | Sector selection, obligation inventory, ADR for choice | 3 | Sector ADR + obligation list |
| 2 | Reference architecture + data-handling/residency design | 5 | Architecture + data-handling spec |
| 3 | Auditability, control-mapping matrix, accountability model | 3 | Control-mapping matrix |
| 4 | Risk taxonomy + HITL flows + grounding/containment spec | 4 | HITL + grounding specs |
| 5 | Toggleable-AI + degraded-mode design | 2 | Toggle spec |
| 6 | Cross-regime portability analysis + package review | 3 | Portability analysis |

## Acceptance criteria

- [ ] Exactly one primary sector is chosen from the menu and justified in an ADR.
- [ ] Sensitive data is classified, with residency, minimization, and retention/deletion controls specified.
- [ ] A control-mapping matrix traces each sector obligation to a concrete architectural control.
- [ ] Audit trail is tamper-evident and queryable and captures who/what/when/why for consequential actions.
- [ ] Accountability is explicit: each automated decision has a named owning human or role.
- [ ] A risk taxonomy defines high-risk output and a HITL approval threshold above which human review is mandatory.
- [ ] Hallucination containment grounds high-risk output in defined vetted sources and refuses/escalates when grounding fails.
- [ ] AI features are toggleable at global, tenant, and per-feature granularity with a defined degraded mode and toggle audit record.
- [ ] The system fails closed when grounding, authorization, or required approval is unavailable.
- [ ] The portability analysis re-derives all five control areas for a second menu sector and classifies each as unchanged/tightened/relaxed/replaced.
- [ ] The analysis concludes with an explicit split of regime-agnostic vs. regime-specific controls.
- [ ] No sector is privileged; the rationale holds for the chosen sector on its own terms.

### Rubric

| Dimension | Weight | What strong work looks like |
|-----------|--------|------------------------------|
| Regulatory translation | 25% | Obligations become concrete controls; the control-mapping matrix is complete and traceable. |
| Data handling and residency | 15% | Classification, minimization, residency, and retention are specific and defensible. |
| Auditability and accountability | 15% | Tamper-evident, reconstructable audit trail with clear human ownership of automated decisions. |
| HITL and risk taxonomy | 15% | Risk tiers and the approval threshold are well-reasoned; fail-closed behavior is consistent. |
| Hallucination containment | 15% | Grounding in vetted sources is enforced; ungrounded high-risk output is refused or escalated. |
| Portability analysis | 15% | Cross-regime re-derivation is rigorous and cleanly separates regime-agnostic from regime-specific controls. |

## Stretch goals

- **Two secondary sectors**: Extend the portability analysis to a second additional sector and surface the controls that differ across all three.
- **Consent and data-subject rights**: Add a consent/erasure flow (e.g., access, correction, deletion requests) and show how the agent and audit log honor it.
- **Red-team the grounding**: Attempt to elicit a confident ungrounded high-risk claim and show the containment holds.
- **Tenant isolation proof**: Demonstrate that one tenant's data and toggles cannot leak into another's.
- **Auditor walkthrough**: Write the script an auditor would follow to reconstruct a specific high-risk decision end to end.

## Submission guidelines

1. Use synthetic data only; never include real regulated records.
2. Keep the regime-agnostic vs. regime-specific distinction explicit throughout, not just in the portability section.
3. Ensure diagrams, the control-mapping matrix, and the specs are mutually consistent.
4. Self-assess against the rubric and record open compliance questions in ADRs.

---

**Ready to start?** Begin with Phase 1: choose your sector and inventory its obligations.
