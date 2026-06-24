# Quiz 1 — Governance, Compliance & Domain Constraints

Knowledge check for [mod-309](../README.md). Answers are at the bottom; try each question before scrolling. Covers all five chapters.

## Questions

### 1. The test for a real control

You write "the platform is committed to fairness and accountability" in your governance doc. Why is this not yet a control?

- A. It uses passive voice.
- B. It maps to no inspectable component and no evidence source — a control you cannot point at in the architecture does not exist.
- C. Fairness is not in scope for ISO/IEC 42001.
- D. It should be in the risk register instead.

### 2. What ISO/IEC 42001 actually is

How is ISO/IEC 42001 best characterized?

- A. A model-level technical test suite for accuracy and bias.
- B. A certifiable AI *management system* standard (clauses 4–10, Annex A reference controls selected by risk) that governs how you run the AI practice, with a Plan-Do-Check-Act cycle.
- C. A U.S. federal regulation enforced by the FTC.
- D. A voluntary risk framework with GOVERN/MAP/MEASURE/MANAGE functions.

### 3. NIST AI RMF — the GOVERN function

In the NIST AI RMF, what is distinctive about the GOVERN function relative to MAP, MEASURE, and MANAGE?

- A. It is the final phase, run after the system ships.
- B. It is optional for low-risk systems.
- C. It is not a phase — it is the always-on culture, policy, accountability, and oversight that runs across the other three functions.
- D. It only applies to generative AI.

### 4. Why agentic systems raise the governance stakes

Which property of an agentic system most directly forces you to *engineer auditability in before launch* rather than retrofit it?

- A. Its latency.
- B. Non-determinism — the same input can produce different actions, so you cannot re-run the agent later to reconstruct what it did.
- C. Its token cost.
- D. The number of plugins it loads.

### 5. The translation method's load-bearing step

In the five-question translation method (data / flow / action / control / evidence), how do you know you have *not* finished translating a requirement?

- A. The requirement lacks a clause number.
- B. The flow (question 2) and the enforcing component (question 4) did not change — the requirement lives only in a policy document.
- C. There is no owner assigned.
- D. The requirement applies to more than one data class.

### 6. Privacy as architecture

A regulation requires data minimization. What is the architecturally correct realization?

- A. Instruct the model in the system prompt to ignore fields it doesn't need.
- B. A field-level scoping layer at the retrieval boundary so the agent context only ever receives the fields its task needs.
- C. Encrypt all fields at rest.
- D. Add a disclaimer to the output.

### 7. Auditability and tamper-evidence

Why must an agent's audit log live in append-only / WORM or hash-chained storage with access separated from the agent's identity?

- A. To reduce storage cost.
- B. So a compromised agent (or an attacker who gains the agent's identity) cannot rewrite the record of what it did — a log the actor can edit is not an audit log.
- C. To make queries faster.
- D. Because non-determinism requires it.

### 8. A plugin, from a risk lens

Which statement best frames what a plugin *is* for governance purposes?

- A. A configuration file with no runtime authority.
- B. New code running with the agent's authority — capable of exfiltration, unauthorized action, silent quality degradation, and unauthorized system change — and therefore a supplier relationship and a life-cycle artifact at once.
- C. A read-only data source that needs no review.
- D. A UI component.

### 9. Plugin versioning

In the plugin lifecycle, approval is of *what*, exactly?

- A. The plugin's name, forever.
- B. A specific signed, pinned version identified by content hash; any change re-enters at the evaluate step.
- C. The plugin author's company.
- D. The trust tier alone.

### 10. The kill switch

Why does the chapter insist the emergency, fleet-wide revocation path be *drilled*, not just documented?

- A. Drilling is required by COPPA.
- B. An undrilled kill switch is a hope, not a control — if revoking a compromised plugin actually requires a redeploy or fails under pressure, you have no containment.
- C. Drilling reduces token cost.
- D. It satisfies the data-minimization requirement.

### 11. FERPA vs. COPPA

A K-12 tutoring agent serves a student under 13 and reads education records. Which is correct?

- A. Only FERPA applies, because the data is education records.
- B. Only COPPA applies, because the user is under 13.
- C. Both typically apply at once (plus state law): FERPA governs disclosure of education records; COPPA governs collection of personal information from under-13s. Design to the union.
- D. Neither applies if a school is involved.

### 12. The same boundary, twice

Why do FERPA's purpose-limitation boundary and COPPA's no-commercial-use boundary end up as the *same* edge in a K-12 architecture?

- A. Coincidence of naming.
- B. Both forbid the regulated student data from flowing to marketing/profiling/training/secondary use — so in the architecture they are the same absent, verified edge from the regulated-data path to any commercial subsystem.
- C. Because both are enforced by the same federal agency.
- D. Because the model provider merges them.

### 13. HITL on student-facing output

Under a consequence-tiered HITL model, which output requires a human decision-maker in the path before it takes effect?

- A. Rephrasing a hint for clarity.
- B. Anything affecting a grade, a permanent record, an IEP/504 plan, a disciplinary outcome, or a parent communication.
- C. Every single token the agent produces.
- D. Only outputs longer than 500 words.

### 14. Hallucination containment for minors

What is the safe *default* behavior for a child-facing tutoring agent facing a high-stakes factual claim it cannot ground in an approved source?

- A. Generate the most plausible answer to keep the student engaged.
- B. Defer — admit uncertainty and route to a human / approved source; bias the system toward "I don't know" on high-stakes factual content.
- C. Search the open web for an answer.
- D. Lower its confidence threshold and answer anyway.

### 15. Fleet governance and accountability

In the fleet RACI, what is the rule for the **Accountable** column?

- A. Every team listed shares accountability equally.
- B. Exactly one Accountable per governance activity, and it is a role with a person attached — not a committee — so the risk traces to one human who signed for it.
- C. The Accountable party is always the on-call SRE.
- D. Accountability rotates weekly.

## Answer key

1. **B** — A control must map to an inspectable component *and* an evidence source; a commitment with neither is aspiration, not a control ([Chapter 1](../01-governance-frameworks.md)).
2. **B** — ISO/IEC 42001 is a certifiable management-system standard (clauses 4–10, Annex A reference controls selected by risk, PDCA cycle), not a technical test suite or a regulation ([Chapter 1](../01-governance-frameworks.md)).
3. **C** — GOVERN is always-on culture/policy/accountability/oversight running across MAP, MEASURE, and MANAGE; it is not a phase ([Chapter 1](../01-governance-frameworks.md)).
4. **B** — Non-determinism means you cannot re-run the agent to learn what it did, so the trace must be emitted at the moment of action ([Chapter 1](../01-governance-frameworks.md), [Chapter 2](../02-regulated-domain-as-architecture.md)).
5. **B** — If the flow and enforcing component did not change, the requirement lives only on paper and is not yet translated ([Chapter 2](../02-regulated-domain-as-architecture.md)).
6. **B** — Minimize at the retrieval boundary with field-level scoping; do not hope the model ignores extra fields ([Chapter 2](../02-regulated-domain-as-architecture.md)).
7. **B** — Tamper-evident, identity-separated storage stops a compromised agent from rewriting its own record; an editable log is not an audit log ([Chapter 2](../02-regulated-domain-as-architecture.md)).
8. **B** — A plugin is new code running with the agent's authority — a supplier relationship and a life-cycle artifact carrying exfiltration, unauthorized-action, quality-degradation, and system-change risk ([Chapter 3](../03-plugin-extension-governance.md)).
9. **B** — Approval is of a specific signed, hash-pinned version; any change re-enters at evaluate ([Chapter 3](../03-plugin-extension-governance.md)).
10. **B** — An undrilled or redeploy-dependent kill switch is not containment; the emergency revocation path must be exercised ([Chapter 3](../03-plugin-extension-governance.md)).
11. **C** — FERPA (disclosure of education records) and COPPA (collection from under-13s) typically both apply, plus state law; design to the union of constraints ([Chapter 4](../04-sensitive-sectors-k12-edtech.md)).
12. **B** — Both regimes forbid the regulated data flowing to commercial/secondary use, so they collapse into one verified absent edge to any marketing/profiling/training subsystem ([Chapter 4](../04-sensitive-sectors-k12-edtech.md)).
13. **B** — Consequence-tiered HITL puts a human decision-maker before anything affecting grades, records, IEP/504s, discipline, or parent comms ([Chapter 4](../04-sensitive-sectors-k12-edtech.md)).
14. **B** — A child-facing agent should fail toward "I don't know" on ungrounded high-stakes facts, deferring to a human or approved source ([Chapter 4](../04-sensitive-sectors-k12-edtech.md)).
15. **B** — Exactly one Accountable per activity, a named role rather than a committee, so the risk traces to one human who signed for it ([Chapter 5](../05-fleet-governance-and-risk-specs.md)).
