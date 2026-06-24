# Chapter 2 — Regulated-Domain Constraints as Architecture

A regulation is a constraint on behavior. An architecture is a set of components, boundaries, and flows. Translation is the architect's core governance skill: turning "the controller shall be able to demonstrate compliance" or "access shall be limited to officials with a legitimate need" into a boundary on a diagram and a log you can produce on demand. This chapter gives you a repeatable method for that translation and works the three obligations that recur in nearly every regulated domain: **data privacy**, **auditability**, and **accountability**.

## The translation method

For each regulatory requirement, answer five questions. The answers *are* the architecture.

```text
1. WHAT DATA?     Which data classes does the requirement govern?
                  (PII, PHI, student education records, payment data)
2. WHAT FLOW?     Where does that data enter, move, rest, and leave?
                  Draw it. The boundary the rule cares about is on this diagram.
3. WHAT ACTION?   Which agent capabilities touch the data or take the
                  regulated action? (read, write, disclose, decide)
4. WHAT CONTROL?  What component enforces the constraint at that point?
                  (boundary, filter, gate, encryption, scoping, log)
5. WHAT EVIDENCE? What does the control emit that proves it ran?
                  (audit log entry, approval record, redaction count)
```

If a requirement does not change the answers to 2 and 4 — the flow and the enforcing component — you have not finished translating it. A requirement that lives only in a policy document is a requirement you cannot demonstrate.

## Data privacy → data-flow boundaries

Privacy regimes (GDPR, HIPAA, FERPA, COPPA, CCPA/CPRA) share a small set of architectural primitives once you strip the legal language:

- **Data minimization** → the agent receives *only* the fields its task needs. Architecturally: a **field-level scoping layer** between your data store and the agent context. An agent helping a student does not need the student's home address or guardian's payroll record in its prompt. Minimize at the retrieval boundary, not by hoping the model ignores extra fields.
- **Purpose limitation** → data acquired for purpose A is not silently reused for purpose B. Architecturally: **tagged data lineage** and tool scopes keyed to purpose; a tool authorized for "tutoring" cannot read records tagged "disciplinary."
- **Consent / lawful basis** → some processing is gated on a recorded consent (especially for minors — Chapter 4). Architecturally: a **consent check** in the request path that fails closed when consent is absent or withdrawn.
- **Data subject rights** (access, deletion, correction) → you must be able to find, return, and delete an individual's data. Architecturally: **per-subject indexing** and a deletion path that reaches every store *including the agent's memory and any vector index* — agent memory is a data store and is in scope.
- **Cross-border / sub-processor limits** → data may not leave a jurisdiction or reach an unapproved processor (a model API is a processor). Architecturally: **region-pinned inference and storage**, and a vetted list of model/tool providers (this is where Chapter 3's plugin governance and your data-processing agreements meet).

The unifying move is the **data-flow boundary**: a place in the architecture where a data class is checked, scoped, redacted, or stopped. Draw your boundaries first; the controls hang off them.

```text
   ┌─────────────┐   raw records    ┌──────────────────┐
   │ system of    │ ───────────────▶ │ scoping +        │  purpose-tagged,
   │ record (SoR) │                  │ redaction layer  │  minimized fields
   └─────────────┘                  └────────┬─────────┘
                                              │
                                  consent / purpose check (fail closed)
                                              │
                                              ▼
                                     ┌──────────────────┐
                                     │ agent context    │  ◀── only what the
                                     │ (prompt + tools) │      task needs
                                     └────────┬─────────┘
                                              │ tool calls (scoped)
                                              ▼
                                     external actions + audit log
```

## Auditability → immutable, queryable logs

Auditability is the ability to **reconstruct what happened and prove it** after the fact. For an agent that planned, called tools, and acted, "what happened" is a chain of decisions and actions — not a single request/response. The architectural requirements:

- **Trace every decision and action.** Each agent step (model call, tool call, tool result, plan revision, handoff) gets a record under a correlating trace ID. This is the same trace infrastructure from observability (mod-305), now load-bearing for compliance. Capture: timestamp, actor (which agent/fleet), input summary, tool invoked, arguments, result, and the pinned model + prompt + plugin versions in effect.
- **Make the log tamper-evident.** Audit logs that the agent (or an attacker who compromises the agent) could rewrite are not audit logs. Use **append-only, write-once storage** (WORM), or hash-chained / signed log entries, with access to the log itself separated from the agent's identity.
- **Retain to the regulatory horizon.** Retention periods are set by the regime (e.g., some education and health records far outlive a single session). Your log lifecycle policy is a compliance artifact, not an ops convenience — and it must coexist with deletion rights (you keep the audit record of an action while deleting the underlying personal data where required, often by storing references rather than raw PII in the log).
- **Make it queryable.** "We have the logs somewhere" fails an audit. You need to answer "show every action this fleet took on subject X in window Y" quickly. Index by subject, actor, action type, and time.

> **Auditability is not free and not retrofittable cheaply.** If you do not emit the trace at the moment of the action, you cannot reconstruct it later. Design the trace schema before launch; non-determinism (Chapter 1) means you cannot re-run the agent to find out what it did.

## Accountability → named owners and decision records

Accountability answers "**who is responsible when this agent acts**?" Regulators and incident responders alike will ask it. Diffuse accountability ("the system did it") is a finding, not an answer. Architecturally and organizationally:

- **A named accountable owner per agent and per fleet.** Not a team — a role with a name attached (covered as RACI in Chapter 5). Encode it in the agent's registry entry so it is discoverable, not buried in a wiki.
- **Decision records for consequential design choices.** When you decide an agent *may* take an action autonomously vs. requiring human approval, that is an accountable decision. Record it as an architecture decision record (ADR): the decision, the rationale, the risk accepted, and who accepted it. The risk a system carries should always trace to a human who signed for it.
- **Human-in-the-loop where the regulation (or your risk tolerance) demands a human decision-maker.** Some regimes restrict *solely automated* decisions with significant effects on a person (e.g., GDPR Article 22 and analogous provisions in the EU AI Act for high-risk systems). Architecturally that means an **approval gate** before the consequential action, with the human's identity and decision captured in the audit log — making accountability and auditability the same record.
- **Traceable authority.** Every tool grant traces to an approval, and every action traces to the grant that authorized it. If an agent took an action it was never granted authority for, that is a containment failure you want to detect, not discover in litigation.

## A worked translation

Requirement (paraphrased FERPA, detailed in Chapter 3 of this domain set): *a school may disclose personally identifiable information from a student's education record to a vendor acting as a "school official" only for the specified function, under the school's direct control, and the disclosure must be subject to recordkeeping.* Run the method:

| Question | Answer for an AI tutoring agent |
| --- | --- |
| What data? | Student education records (grades, IEP data, identifiers). |
| What flow? | SoR → scoping layer → agent context → tutoring output; no path to marketing, other students, or untagged purposes. |
| What action? | `read_student_record` (scoped to the one student in session); no `disclose_externally`. |
| What control? | Field-level scoping + purpose tag "instructional"; tool whitelist excludes external disclosure; region-pinned inference under a data-processing agreement. |
| What evidence? | Audit log of every record access by student, time, and purpose; the DPA naming the vendor as a school official under district control. |

The legal sentence became a diagram boundary, a tool scope, and a log query. That is the whole job.

## Key takeaways

- Translate every regulatory requirement through five questions — data, flow, action, control, evidence; if the flow and enforcing component do not change, you have not translated it.
- **Privacy** becomes data-flow boundaries: minimization at the retrieval edge, purpose-tagged lineage, fail-closed consent checks, per-subject deletion that reaches agent memory and vector indexes, and region/sub-processor pinning.
- **Auditability** becomes tamper-evident, append-only, queryable traces of every decision and action — designed before launch because non-determinism makes it unreconstructable after the fact.
- **Accountability** becomes named owners, decision records (ADRs), HITL gates on consequential actions, and authority that traces from action → grant → approval → a human who signed for the risk.
