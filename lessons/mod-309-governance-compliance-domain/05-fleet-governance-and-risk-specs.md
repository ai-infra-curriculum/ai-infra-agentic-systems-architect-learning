# Chapter 5 — Governance, Accountability & Risk-Control Specs for Agent Fleets

A single agent is a governance problem you can hold in your head. A **fleet** — dozens or hundreds of agents, many versions, multiple owners, shared tools and plugins, running continuously across tenants and use cases — is not. At fleet scale, governance must become a set of **specifications and registries** that hold without anyone remembering the details: an operator runs against them, an auditor reviews them, and a new team onboarding an agent inherits them. This chapter is about producing those artifacts. It is the synthesis of the module: everything from Chapters 1–4 lands in a small set of documents and tables that govern the fleet.

## What "governing a fleet" actually requires

The fleet adds problems a single agent does not have:

- **Heterogeneity.** Agents differ in risk: a read-only internal summarizer and a student-facing tutor with write access cannot share one control regime. Governance must be **tiered**.
- **Scale of accountability.** Hundreds of agents need hundreds of owners — discoverable, not tribal knowledge. A registry, not a wiki page.
- **Shared blast radius.** A shared tool, plugin, or base prompt is a single point of failure across the fleet. Fleet-wide controls (kill switches, version pins) matter more than per-agent ones.
- **Continuous change.** New agents, new versions, new plugins land constantly. Governance is a **process with gates**, not a launch-day review.
- **Aggregate risk.** Each agent may be acceptable while the fleet's combined data access or action volume is not. Someone must own the aggregate view.

The specifications below are the response. Treat them as living documents under version control, owned by the AIMS owner (Chapter 1, ISO/IEC 42001 clause 5), reviewed on a cadence.

## Artifact 1 — The agent registry

The system of record for "what agents exist and who is accountable." Every agent in the fleet has an entry; an agent not in the registry is not authorized to run (the runtime checks the registry, exactly as it does for plugins in Chapter 3).

| Field | Purpose |
| --- | --- |
| Agent ID / name | Stable identifier. |
| Accountable owner | Named role (the **A** in the RACI below). |
| Purpose & use case | What it does; ties to its impact assessment. |
| Risk tier | Tier 1–4 (below); sets the applicable controls. |
| Data classes touched | Drives privacy obligations (Chapter 2). |
| Tools / plugins granted | Each grant traces to an approval (Chapters 2–3). |
| HITL requirements | Which actions need a human gate (Chapter 4). |
| Model + prompt versions | Pinned, for reproducibility and audit. |
| Status | Active / paused / retired. |
| Impact assessment ref | Link to the completed assessment. |

## Artifact 2 — The risk-tier model

Tiering is what makes fleet governance affordable: it concentrates rigor where risk is. Define tiers by **impact of a wrong or malicious action**, then attach a control set to each.

```text
TIER 4 — Critical:   acts on minors/regulated subjects, or irreversible/
                     high-impact actions (grades, records, payments, deletes).
                     → mandatory HITL on consequential actions; full audit;
                       highest review cadence; legal/privacy sign-off.

TIER 3 — High:       writes to systems of record, or touches regulated data
                     without minor/irreversible impact.
                     → scoped write tools; audit; named-owner sign-off; eval gate.

TIER 2 — Moderate:   reads regulated/internal data; no external action.
                     → read scoping; audit; standard eval gate; owner approval.

TIER 1 — Low:        read-only on non-sensitive data; ephemeral output.
                     → lightweight review; monitoring; auto-approve under policy.
```

The point is proportionality: gate Tier 4 hard, let Tier 1 move fast. Uniform heavy governance is over-governance — it trains teams to misclassify down to escape it. Make tier assignment a reviewed decision, and audit a sample of self-assigned tiers.

## Artifact 3 — The RACI (accountability matrix)

Accountability that names a team is not accountability. RACI assigns, per governance activity, who is **R**esponsible (does the work), **A**ccountable (owns the outcome — exactly one), **C**onsulted, and **I**nformed. A worked slice:

| Activity | Responsible | Accountable | Consulted | Informed |
| --- | --- | --- | --- | --- |
| Approve a new Tier 4 agent | Agent team lead | Head of AI platform | Legal, Privacy, Security | Fleet operators |
| Approve a plugin (Chapter 3) | Platform eng | Plugin governance owner | Security | Affected agent owners |
| Run an impact assessment | Agent owner | Agent owner | Privacy | AIMS owner |
| Respond to a safety incident | On-call SRE | Incident commander | Owner, Legal | Leadership |
| Set / change a risk tier | Agent owner | AIMS owner | Risk lead | Platform team |
| Execute a fleet-wide kill switch | On-call SRE | Incident commander | — | All owners |

Rule: **exactly one Accountable per row**, and it is a person/role, not a committee. The risk a system carries traces to one human who signed for it.

## Artifact 4 — The risk register

The MAP/MEASURE output (Chapter 1, NIST AI RMF) as a living table. Source risks from the NIST GenAI Profile and your impact assessments. Each row:

| Field | Example |
| --- | --- |
| Risk ID | R-014 |
| Description | Tutoring agent asserts ungrounded fact to a student. |
| Affected parties | Students (minors). |
| Likelihood / impact | Medium / High → composite High. |
| Mitigating control(s) | Grounded retrieval; output-side claim check; HITL on high-stakes (Chapter 4). |
| Residual risk | Low. |
| Owner | Tutoring product owner. |
| Status / review date | Open; reviewed quarterly. |

The register's job is to make residual risk **explicit and owned**. A risk with no owner and no control is the finding an incident review will start from.

## Artifact 5 — The control catalog

The bridge back to Chapter 1: the master controls-mapping table for the fleet. Each control: an ID, the framework clause(s) it satisfies (42001 Annex A / NIST RMF subcategory), the architectural component that realizes it, which risk tiers it applies to, the owner, and the **evidence source** that proves it operates. This is the artifact an auditor reads first, and the one that lets you produce a statement-of-applicability per tenant or per certification scope.

| Control ID | Framework ref | Realization | Applies to | Evidence |
| --- | --- | --- | --- | --- |
| C-01 | 42001 A (life cycle); NIST MANAGE | Gated promotion pipeline + eval suite | All tiers | Pipeline config; eval results per release |
| C-07 | NIST MANAGE; 42001 Cl. 10 | Fleet-wide kill switch | T3–T4 | Drill records; revocation logs |
| C-12 | 42001 A (third party) | Plugin registry + signing (Chapter 3) | All tiers | Approval + signature-verification logs |
| C-15 | 42001 Cl. 6; GDPR/FERPA | HITL gate on consequential actions | T4 (T3 partial) | Approval logs with human identity |

## Artifact 6 — The fleet incident & change process

Specifications govern the steady state; the incident and change processes govern the moments that matter.

- **Change control with gates.** New agent, new version, new plugin, tier change → defined gate (eval pass, owner sign-off appropriate to tier, registry update). No silent promotion to production.
- **Incident runbook.** Detection → triage → contain (pause agent / revoke plugin / kill-switch the fleet) → eradicate → recover → review. Map to NIST MANAGE (respond and recover). The kill switch must be *drilled*, not just documented.
- **Continual improvement loop.** Incidents and audit findings feed corrective actions back into controls and specs — the Plan-Do-Check-Act cycle of ISO/IEC 42001 clause 10. Governance that does not learn decays.

## Putting it together

The fleet governance specification is the union of these six artifacts: **registry** (what exists, who owns it), **tier model** (how much rigor), **RACI** (who decides), **risk register** (what could go wrong and who owns it), **control catalog** (how each obligation is realized and evidenced), and **incident/change process** (how change and failure are handled). Each ties back to the framework mapping of Chapter 1, the data/audit/accountability boundaries of Chapter 2, the plugin lifecycle of Chapter 3, and the sensitive-sector controls of Chapter 4. Together they let one architect govern a fleet they could never hold in their head — and let an auditor verify it without taking anyone's word.

## Key takeaways

- Fleet governance must be **specifications and registries**, not memory: heterogeneity, scale of accountability, shared blast radius, continuous change, and aggregate risk all defeat per-agent, launch-day review.
- The six load-bearing artifacts are the **agent registry**, the **risk-tier model**, the **RACI**, the **risk register**, the **control catalog**, and the **incident/change process** — each living, owned, version-controlled, and tracing back to Chapters 1–4.
- **Tier rigor to impact** (gate Tier 4 hard, let Tier 1 move fast) — uniform heavy governance is over-governance that drives misclassification; assign exactly one Accountable owner per governance activity.
- The control catalog with an **evidence column** is what an auditor reads first and what produces a per-tenant statement of applicability; the incident process must include a *drilled* fleet-wide kill switch and a continual-improvement loop that feeds findings back into the controls.
