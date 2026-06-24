# exercise-05: Governance & Accountability Spec

**Estimated effort:** 3 hours

## Objective

Produce the fleet-level governance, accountability, and risk-control specification for an agent fleet: the six load-bearing artifacts — registry, risk-tier model, RACI, risk register, control catalog, and incident/change process — that let one architect govern a fleet they cannot hold in their head, and let an auditor verify it without taking anyone's word. This is the capstone deliverable of the module; it integrates exercises 01–04.

## Background

This exercise covers material from:

- [Chapter 5 — Governance, Accountability & Risk-Control Specs for Agent Fleets](../05-fleet-governance-and-risk-specs.md)
- All prior chapters — the framework mapping (Ch. 1), data/audit/accountability boundaries (Ch. 2), plugin lifecycle (Ch. 3), and sensitive-sector controls (Ch. 4) all land here.

Fleet governance is specifications and registries, not memory. Each artifact must be living, owned, and traceable back to a control.

## Scenario

The **TutorMesh** platform now runs a fleet of agents:

- `tutor` — student-facing, touches education records, serves minors (Tier 4).
- `lesson-drafter` — teacher-facing, reads curriculum, drafts lesson plans (Tier 2).
- `progress-summarizer` — writes notes to the teacher dashboard, drafts parent-facing summaries (Tier 3).
- `usage-reporter` — read-only internal analytics over de-identified usage (Tier 1).

Shared across the fleet: a curriculum knowledge base, the plugin registry from exercise-02, and a base safety prompt. Produce the governance spec that governs all four at once.

## Tasks

### 1. The agent registry

- Build the registry with one row per agent: ID, accountable owner (named role), purpose, risk tier, data classes touched, tools/plugins granted, HITL requirements, pinned model/prompt versions, status, and impact-assessment reference.
- State the runtime rule: an agent not in the registry is not authorized to run.

### 2. The risk-tier model

- Define Tiers 1–4 by **impact of a wrong or malicious action**, and attach a control set to each tier (HITL, audit depth, review cadence, sign-off level).
- Place the four agents in tiers and justify each placement.

### 3. The RACI

- Build the accountability matrix for at least six governance activities (approve a new Tier 4 agent, approve a plugin, run an impact assessment, respond to a safety incident, set/change a risk tier, execute a fleet-wide kill switch).
- Enforce the rule: **exactly one Accountable per row**, and it is a role with a person, not a committee.

### 4. The risk register

- Build a living risk register: at least six risks drawn from the NIST GenAI Profile categories and your prior exercises, each with description, affected parties, likelihood/impact, mitigating control(s), residual risk, owner, and review date.
- Include at least one risk whose affected party is a minor.

### 5. The control catalog and incident/change process

- Build the control catalog: each control with an ID, the framework reference it satisfies (42001 Annex A / NIST RMF), its architectural realization, the tiers it applies to, the owner, and the **evidence source**.
- Specify the change-control gates (new agent / new version / new plugin / tier change) and the incident runbook (detection → triage → contain → eradicate → recover → review), including a **drilled** fleet-wide kill switch.

## Starter guidance

Use this governance-spec outline. It is the union of six artifacts; fill every one.

```text
# TutorMesh Fleet Governance Specification
# Owner: <AIMS owner role>   Review cadence: <e.g., quarterly>   Version: <n>

## 1. Agent registry
| Agent | Owner (role) | Purpose | Tier | Data classes | Tools/plugins | HITL | Model+prompt ver | Status | Impact assess ref |

## 2. Risk-tier model
TIER 4 critical → mandatory HITL on consequential actions; full audit; highest cadence; legal sign-off
TIER 3 high     → scoped write tools; audit; named-owner sign-off; eval gate
TIER 2 moderate → read scoping; audit; standard eval gate
TIER 1 low      → lightweight review; monitoring; auto-approve under policy

## 3. RACI (exactly one Accountable per row)
| Activity | Responsible | Accountable | Consulted | Informed |

## 4. Risk register
| Risk ID | Description | Affected parties | Likelihood/Impact | Control(s) | Residual | Owner | Review date |

## 5. Control catalog
| Control ID | Framework ref | Realization | Applies to tiers | Owner | Evidence source |

## 6. Incident & change process
Change gates: new agent | new version | new plugin | tier change → <gate>
Incident runbook: detect → triage → contain (pause/revoke/kill) → eradicate → recover → review
Kill switch: fleet-wide, fast, logged, DRILLED
Continual improvement: findings → corrective actions → controls (PDCA, 42001 Cl. 10)
```

## Acceptance criteria

You can demonstrate that:

- All **six artifacts** exist and are populated for the four-agent fleet, each living and owned.
- The registry has a named accountable owner per agent and the "not in registry = not authorized" rule is stated.
- The tier model is **proportionate** (Tier 4 gated hard, Tier 1 moves fast) and the four agents are placed with justification.
- The RACI has **exactly one Accountable** per activity, named as a role.
- The risk register makes residual risk explicit and owned, including at least one minor-affecting risk; the control catalog has an **evidence column** for every control and correct framework references.
- The incident/change process includes defined gates and a **drilled** fleet-wide kill switch, with a continual-improvement loop.

## Reflection

In `NOTES.md`:

1. Where did an individually-acceptable agent contribute to an **aggregate** fleet risk (combined data access or action volume) that no single registry row captured? How did you surface it?
2. Which control is the highest-leverage fleet-wide one (largest shared blast radius), and why?
3. Trace one risk from your register all the way to the framework clause it satisfies and the evidence an auditor would request. Where, if anywhere, did the chain have a gap?

## Stretch goals

- Generate a per-tenant statement of applicability for one district from the control catalog.
- Add an aggregate-risk dashboard spec: the fleet-level metrics (total regulated-data access, action volume by tier, open high risks) the AIMS owner reviews.
- Write the management-review (42001 Clause 9.3) agenda that keeps this entire specification current, and the trigger conditions for an off-cycle review.
