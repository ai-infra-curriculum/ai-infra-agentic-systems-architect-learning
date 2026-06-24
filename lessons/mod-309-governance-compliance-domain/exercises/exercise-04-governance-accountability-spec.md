# exercise-04: Governance and Accountability Spec

**Estimated effort:** 3 hours

## Objective

Author the **fleet-level governance specifications** for an agent fleet: a **lineage** spec, an **audit-trail** spec, and a **human-accountability** spec (decision-gate roles, fleet RACI, intervention paths, contestability). The deliverable is precise enough that an engineer can implement it and an auditor can test it — culminating in a worked incident walk-through proving the specs make a failure traceable.

This is an **architecture/specification** exercise. You produce the specs and a risk-control catalog, not running code.

## Background

This exercise covers material from:

- [Chapter 5 — Fleet Governance and Accountability Specs](../05-fleet-governance-and-accountability-specs.md)
- [Chapter 2 — Regulated Domain as Architecture](../02-regulated-domain-as-architecture.md) — the auditability and accountability primitives these specs operationalize.

## The scenario

> An agent fleet runs continuously: many agents, several live code and tool versions at once, internal teams' tools, thousands of runs a day. It takes real-world actions (writes records, notifies external parties). It is deployed under a regulated regime (pick one of the peer regimes from Chapter 3 — it sets the gate authority and retention parameters, but the specs themselves are regime-neutral in structure).

Write the specs that make this fleet governable.

## Tasks

### 1. Author the lineage spec

- Specify, per [Chapter 5](../05-fleet-governance-and-accountability-specs.md), the provenance chain recorded for any output/decision: agent version, model version (+ provider/region), prompt version, tool versions (from the registry), data class/source, run id, and active regime profile.
- Specify that the lineage record is itself **minimized** (references data by class/source, not raw content) and how it links to the durable run history of [mod-308](../../mod-308-deployment-durable-execution/README.md).

### 2. Author the audit-trail spec

Fix the four things a real audit trail requires:

- **What is recorded** — which *consequential* events (actions, decisions incl. the model's, human gate decisions, policy enforcements, version/config changes).
- **The record schema** — a fixed field list (timestamp, run id, agent+version, actor, event type, subject data class, outcome, rationale).
- **Immutability/integrity** — append-only / tamper-evident, riding the durable history.
- **Retention + queryability** — per-regime retention window and the query axes (by subject, run, agent, time, action type).
- State the **auditor's acid-test query** your spec can answer.

### 3. Author the human-accountability spec

- **Decision gates**: where HITL gates sit (by consequence) and the accountable role at each, parameterized by the active regime.
- **Fleet RACI**: a Responsible/Accountable/Consulted/Informed map for the fleet's *behavior*.
- **Intervention**: the kill-switch / per-agent-disable / tool-revocation paths and who may invoke them.
- **Contestability**: the appeal path for an affected individual and the named handler.

```text
   decision gates : action → accountable role (per regime)
   fleet RACI     : behavior → R / A / C / I
   intervention   : kill-switch / disable / revoke → who may invoke
   contestability : affected-party appeal → named handler
```

### 4. Build the risk-control catalog

- For each top fleet risk (drawn from RMF's Map in exercise-01), one row: risk → control → component it lives in → spec that evidences it → named owner.

| Risk | Control | Component | Evidencing spec | Owner |
| --- | --- | --- | --- | --- |

- Any row with a missing component or owner is an open finding — list those.

### 5. Walk an incident through the specs

- Incident: *a fleet agent took a wrong, harmful action affecting an individual.* Show how **lineage**, **audit trail**, and **human-accountability** together make the incident traceable: which version/model/tools/data/run, what was decided and by whom, who is accountable, and the contestability + revocation paths. Contrast with what the investigation looks like *without* the specs.

## Starter guidance

The three interlocking specs:

```text
   LINEAGE (provenance of an artifact)
     └─ run id ──┐
   AUDIT TRAIL (sequence of decisions/actions) ── linked by run id
     └─ each gate decision writes the decider ──┐
   HUMAN-ACCOUNTABILITY (who owns each gate + the fleet)
```

An audit-record schema to adopt and refine:

```text
   { ts, run_id, agent+version, actor(agent|human:role),
     event_type, subject_data_class, outcome, rationale? }
   append-only · per-regime retention · queryable by subject/run/agent/time
```

You do **not** re-derive the framework mapping (exercise-01) or the regime parameters (exercise-02) — *consume* them: the risk list comes from RMF Map, the gate authority and retention come from the regime profile.

## Acceptance criteria

You can demonstrate that:

- The lineage spec records the full provenance chain, is minimized, and links to the durable run history and the tool registry.
- The audit-trail spec fixes what-is-recorded, a concrete schema, immutability, and per-regime retention/queryability — and you state the auditor's acid-test query it answers.
- The human-accountability spec names roles for each gate, a fleet RACI, intervention paths with who-may-invoke, and a contestability path with a named handler.
- The risk-control catalog ties each top risk to a control, component, evidencing spec, and owner, with open findings listed.
- The incident walk-through is traceable end-to-end using the three specs, and the without-specs contrast is explicit.

## Reflection

In `NOTES.md`:

1. Your fleet has several live versions at once. Show a concrete case where *omitting agent version from lineage* would make an incident unanswerable.
2. "We have audit logging" vs. your spec — name three things your spec pins down that ordinary application logging does not.
3. Accountability without intervention power is hollow. Trace one accountable role and show they actually *can* stop/override the relevant behavior.

## Stretch goals

- Design the **tamper-evidence** mechanism concretely (hash-chaining or external anchoring) and argue why append-only-in-a-DB is insufficient against a privileged operator.
- Add a **periodic governance review** cadence: who reviews the risk-control catalog, how often, and what triggers an out-of-cycle review.
- Reconcile lineage retention with a regime **deletion-right**: how do you keep the audit trail intact while honoring erasure of the underlying data subject? (Tie to the retention primitive of Chapter 2.)
