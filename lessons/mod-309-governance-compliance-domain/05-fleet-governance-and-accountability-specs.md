# Chapter 5 — Fleet Governance and Accountability Specs

The previous chapters governed pieces: a framework mapped onto the architecture, regulatory primitives turned into components, regimes reconciled, the extension surface put under a lifecycle. This chapter assembles them into the artifacts that govern a **fleet** — many agents, many versions, many tools, running continuously and taking real-world actions — and that an operator runs against and an auditor reviews.

At fleet scale, governance is not a property you assert; it is a set of **specifications** you write down and hold the system to. Three of them carry the load: **lineage** (where did this output / decision / data come from?), **audit trails** (what happened, provably?), and **human-accountability points** (who is responsible?). These are the same accountability and auditability primitives from [Chapter 2](02-regulated-domain-as-architecture.md), now expressed as concrete specs that name owners, fields, retention, and replayability. A control you described in prose but cannot point an auditor at is not a control; a spec is how you make it pointable.

> **Specs, not vibes.** "We have audit logging" is a vibe. "Every consequential action emits an immutable record with these eight fields, retained N years, replayable on demand, owned by this role" is a spec. The architect's deliverable at this layer is the spec — precise enough that an engineer can implement it and an auditor can test it.

## What a fleet adds that a single agent does not

A single governed agent is tractable. A fleet introduces governance problems that only exist *because* it is a fleet:

- **Many versions live at once.** Per the in-flight-versioning of [mod-308](../mod-308-deployment-durable-execution/README.md), several code and tool versions run concurrently. Lineage must record *which version* produced a given output — "the agent did X" is useless if you cannot say which agent at which version.
- **Emergent multi-agent behavior.** Agents calling agents produce outcomes no single agent's review anticipated. Accountability must attach to the *fleet's* behavior and to the orchestration, not only to individual agents.
- **Continuous, autonomous action.** The fleet acts around the clock with no human in most loops. Accountability cannot mean "a human approved each action"; it must mean "a named human is responsible for the fleet's behavior and there are defined points where humans own specific decisions."
- **Scale of the audit surface.** Thousands of runs a day across regimes. The audit trail has to be *queryable* and *retained per regime* (Chapters 2–3), not just written somewhere.

The three specs below are the architect's response to exactly these problems.

## Spec 1 — Lineage

Lineage answers *provenance*: for any output, decision, or stored datum, where did it come from and through what? At fleet scale this is what makes an incident investigable and a regulator's question answerable.

A lineage spec defines, for each tracked artifact, the chain it must record:

```text
   OUTPUT / DECISION
     ├─ which agent + agent version (code build id)
     ├─ which model + model version (and provider/region)
     ├─ which prompt / system version
     ├─ which tools called, at which versions  (from the Ch.4 registry)
     ├─ which data / retrieved context (by class + source, minimized)
     ├─ which run id  (ties to the durable history, mod-308)
     └─ which regime profile was in force  (from Ch.3)
```

The architect specifies:

- **Model and data lineage.** The model version (and provider/region, for residency) and the data sources/classes behind an output — so you can answer "was this decision made by a model and data we had approved?" and "did this output derive from data it should not have touched?"
- **Tool and version lineage.** Which tool versions participated, drawn from the immutable registry of [Chapter 4](04-extension-and-tool-governance.md) — so a drifted or revoked tool's blast radius is enumerable after the fact.
- **Run lineage.** A stable run id linking the output back to the durable event history of [mod-308](../mod-308-deployment-durable-execution/README.md), which *is* the replayable record of what the run did.
- **Minimization in the lineage record itself.** Lineage records reference data by class and source, not by copying raw regulated content — the lineage store is a data flow subject to the same privacy primitive.

Lineage is what turns "the agent produced a wrong/harmful output" from an unanswerable mystery into a traceable chain: which version, which model, which tools, which data, which run.

## Spec 2 — Audit trail

Where lineage is about *provenance of an artifact*, the audit trail is about *the sequence of what happened* — the tamper-evident record of decisions and actions across the fleet. It operationalizes the auditability primitive of [Chapter 2](02-regulated-domain-as-architecture.md) into a concrete, testable spec.

An audit-trail spec fixes four things:

- **What is recorded.** Every *consequential* event: an action taken (tool call with minimized args), a decision made (including the model's), a human decision at a gate (who, what, why), a policy enforcement (a guardrail block, a scope denial), a configuration or version change. Not every log line — the *decisions and actions* that a regulator or an incident responder needs.
- **The record's fields.** A fixed schema: timestamp, run id, agent + version, actor (agent or named human), event type, the (minimized) subject data class, the decision/outcome, and the rationale where applicable. A fixed schema is what makes the trail *queryable*.
- **Immutability and integrity.** Append-only or tamper-evident (hash-chained / write-once), riding the durable history where possible so it is replayable and cannot be quietly edited — including by privileged operators.
- **Retention and queryability per regime.** The trail is retained for each regime's required window (Chapter 3's per-class schedules) and is queryable along the axes auditors actually use: by data subject, by run, by agent, by time window, by action type.

The audit trail's acid test is the auditor's question: *"Show me every consequential action this fleet took on this data subject in this period, who or what decided each, and prove the record was not altered."* If the architecture can answer that on demand, the trail is real; if not, it is aspirational logging wearing a compliance label.

## Spec 3 — Human-accountability points

Autonomy does not remove human accountability — it *relocates* it. A fleet that acts continuously cannot put a human in every loop, so the architecture must define the specific points where a named human/role *is* accountable, and make those points real.

A human-accountability spec defines:

- **Decision-gate accountability.** Where the durable HITL gates of [mod-308](../mod-308-deployment-durable-execution/README.md) sit, placed by consequence, and *which role* is the accountable decider at each — the clinician, the compliance officer, the records officer, the educator/guardian, depending on the active regime (Chapter 3). The gate records the decider's identity in the audit trail.
- **Operational accountability (the fleet RACI).** A Responsible / Accountable / Consulted / Informed map for the fleet's *behavior* — who owns it running correctly, who is accountable when it harms, who is consulted on changes, who is informed. This answers the EU AI Act's human-oversight and NIST RMF's Govern obligations at the *fleet* level, not the per-action level.
- **Escalation and override paths.** How a human stops, overrides, or rolls back the fleet — a kill switch, a per-agent disable, the tool-revocation path of [Chapter 4](04-extension-and-tool-governance.md). Accountability without the *power to intervene* is hollow; the accountable human must be able to act.
- **Contestability.** For decisions affecting individuals, a path to question or appeal an automated outcome, with a named human who handles it. This is where high-risk-tier obligations (Chapter 1) become a concrete component.

```text
   HUMAN-ACCOUNTABILITY MAP (per fleet)
   ───────────────────────────────────────────────────────────
   decision gates   : action → accountable role (per regime profile)
   fleet RACI       : behavior → R / A / C / I owners
   intervention     : kill-switch / disable / revoke (who may invoke)
   contestability   : affected-party appeal path → named handler
   ───────────────────────────────────────────────────────────
   every entry names a ROLE, and every gate writes the decider
   into the AUDIT TRAIL (Spec 2), linked by run id to LINEAGE (Spec 1)
```

## Assembling the governance package

The three specs are not independent documents — they interlock, and the architect ships them as one **governance package** for the fleet:

- **Lineage** gives you the provenance chain for any artifact.
- **The audit trail** records the sequence of decisions/actions, each linked by run id to its lineage.
- **Human-accountability points** name who owns each gate and the fleet overall, and every gate decision lands in the audit trail.

Around these three sit the supporting artifacts the earlier chapters produced: the clause-to-component map (Chapter 1), the primitive-to-architecture translations (Chapter 2), the convergence/divergence map and regime profiles (Chapter 3), and the extension registry and lifecycle (Chapter 4). Together they are what an auditor reviews and an operator runs against — the difference between a platform that *is* governable and one that merely *claims* to be.

A useful completeness check is a **risk-control catalog**: for each top fleet risk (from RMF's Map), the control that treats it (Manage), the component it lives in, the spec that evidences it, and the named owner. A risk with no control, or a control with no component or owner, is an open finding — the same test the whole module has applied, now at fleet scale.

## Worked judgment

*An incident: a fleet agent emailed a wrong, harmful recommendation to a customer.* With the three specs in place, the response is mechanical, not forensic archaeology. **Lineage** names the agent version, model version, prompt version, the tool that sent the email, the data behind the recommendation, and the run id. **The audit trail** replays the run's decisions and shows whether a gate was hit and what was decided. **Human-accountability** names who owned that gate (or flags that there should have been one), who is accountable for the fleet, and provides the contestability path for the affected customer — plus the revocation path if a drifted tool was implicated. Without the specs, this is an unanswerable incident; with them, it is a traceable one with named owners. That is the entire value proposition of fleet governance.

## Key takeaways

- At fleet scale, governance is a set of **specifications** you hold the system to, not a property you assert. The three load-bearing specs are **lineage**, **audit trail**, and **human-accountability points** — the Chapter 2 primitives made concrete with owners, fields, retention, and replayability.
- **Lineage** records provenance — agent/model/prompt/tool versions, data class/source, run id, and active regime — so any output is traceable. It rides the durable history (mod-308) and the immutable tool registry (Chapter 4).
- **The audit trail** is the tamper-evident, fixed-schema, per-regime-retained record of consequential decisions and actions; its test is the auditor's "show me every action on this subject, who decided, prove it wasn't altered."
- **Human-accountability points** relocate (not remove) human responsibility: per-regime decision-gate roles, a fleet RACI, intervention/override power, and contestability. The three specs interlock into a **governance package** — the difference between a fleet that is governable and one that only claims to be.
