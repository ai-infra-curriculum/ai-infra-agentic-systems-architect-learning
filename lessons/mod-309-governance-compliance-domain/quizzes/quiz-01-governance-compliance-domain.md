# Quiz 1 — Governance, Compliance & Regulated-Domain Architecture

Knowledge check for [mod-309](../README.md). Answers are at the bottom; try each question before scrolling. Covers all five chapters. The framing throughout is **horizontal-first**: governance as a general discipline, with regulated sectors treated as equal peers.

## Questions

### 1. The test for a real control

You write "the platform is committed to fairness and accountability" in your governance doc. Why is this not yet a control?

- A. It uses passive voice.
- B. It maps to no inspectable component, log, gate, or owner — a control you cannot point at in the architecture does not exist.
- C. Fairness is out of scope for ISO/IEC 42001.
- D. It belongs in the risk register, not the governance doc.

### 2. What ISO/IEC 42001 actually is

How is ISO/IEC 42001 best characterized?

- A. A model-level technical test suite for accuracy and bias.
- B. A certifiable AI *management system* (AIMS) standard — policies, roles, an Annex A control set, run on a Plan-Do-Check-Act cycle — that governs *how an organization runs AI*, not the model's internals.
- C. A binding regulation enforced by a government agency.
- D. A U.S.-only voluntary framework with GOVERN/MAP/MEASURE/MANAGE functions.

### 3. The three frameworks have distinct jobs

You are governing a new agent. Which statement correctly matches each horizontal framework to its job?

- A. All three answer the same question; pick whichever your auditor prefers.
- B. The EU AI Act sets the obligation *tier*; NIST AI RMF gives the risk *process* (Govern/Map/Measure/Manage); ISO/IEC 42001 gives the *management system* to run it repeatably and prove it.
- C. ISO 42001 sets the risk tier; the EU AI Act gives the four functions; NIST RMF certifies the org.
- D. NIST RMF is binding law; the EU AI Act is voluntary; ISO 42001 is sector-specific.

### 4. NIST AI RMF — the Govern function

What is distinctive about the **Govern** function relative to Map, Measure, and Manage?

- A. It is the final phase, run after the system ships.
- B. It is optional for low-risk systems.
- C. It is cross-cutting — the always-on culture, roles, accountability, and risk tolerance that wraps the other three functions.
- D. It applies only to generative AI.

### 5. EU AI Act risk tiers are horizontal

An agent that screens job applicants, one that assists medical triage, and one that scores creditworthiness can all land in the EU AI Act's **high-risk** tier. What does this illustrate?

- A. The Act only regulates HR, healthcare, and finance.
- B. Risk tiers are domain-spanning: systems in unrelated sectors can inherit the *same* structural obligations (risk management, data governance, logging, human oversight, accuracy) — a common obligation set the architect builds against before any sector-specific rule.
- C. High-risk means the system is prohibited.
- D. Each sector defines its own incompatible tiers.

### 6. Why lead horizontal before sector

Why does the module insist on mapping the horizontal frameworks *before* reaching for any specific statute?

- A. Statutes are not legally binding.
- B. Leading with one sector bakes that sector's assumptions into the architecture's bones; leading horizontally builds a platform that is *governable in general*, which you then parameterize per regime.
- C. Horizontal frameworks override all national law.
- D. It is faster to ignore the statutes entirely.

### 7. The five primitive constraints

Per Chapter 2, almost every data regulation — across healthcare, finance, public sector, and edtech — decomposes into the same five primitives. Which set is correct?

- A. Encryption, authentication, authorization, rate-limiting, backups.
- B. Privacy, residency, auditability, accountability, retention.
- C. Latency, throughput, cost, availability, durability.
- D. Consent, contracts, copyright, liability, insurance.

### 8. Translating a privacy constraint

Architecturally, a *privacy* constraint most directly produces which elements?

- A. A faster inference endpoint.
- B. Data-flow boundaries with classification at ingest and minimization (mask/tokenize/redact) before data reaches a component — including the model prompt, tool calls, and logs — that does not need it in the clear.
- C. A longer retention window for all data.
- D. A single shared database for all data classes.

### 9. Auditability vs. ordinary logging

What distinguishes a real *audit trail* from ordinary application logging?

- A. It is stored in JSON.
- B. It is tamper-evident/immutable, captures consequential *decisions and actions* (not just errors) on a fixed schema, is retained for the regime's window, and is queryable by subject/run/agent/time.
- C. It is written at DEBUG level.
- D. It is kept only in memory for speed.

### 10. Convergence vs. divergence

You must deploy one agent under healthcare, finance, public-sector, and edtech regimes. What is the correct architectural strategy?

- A. Fork the platform into four sector-specific builds.
- B. Build a convergent core (classify, minimize, audit-log, accountability gate, retention lifecycle, access policy) once, tuned by a per-regime profile; isolate the small set of structural deltas as pluggable components activated per regime.
- C. Pick the strictest sector and apply only its rules everywhere, ignoring the others.
- D. Apply no controls until a regulator complains.

### 11. Parameter vs. structural divergence

A regime requires *verifiable consent for minors* that the other peer regimes do not. How should this be handled?

- A. As a parameter on the convergent core, like a retention schedule.
- B. As a structural divergence — a pluggable component built once and *activated only for the regimes that require it* — because it is a control the others have no analog for.
- C. By forking the whole platform for that regime.
- D. By ignoring it, since most regimes do not require it.

### 12. Governing the extension surface

Why is an agent's tool/extension registry a first-class governance concern?

- A. Tools are purely cosmetic.
- B. An agent's capabilities *are* its tools; each tool expands the action surface and is a data-egress boundary, so an ungoverned tool registry makes an otherwise-safe agent unsafe.
- C. Only the base model needs governing; tools are out of scope.
- D. Third-party tools are automatically safe if they are popular.

### 13. The extension lifecycle

What is the correct lifecycle every extension (third-party *and* internal) must pass through and stay inside?

- A. Install → forget.
- B. Evaluate → version → approve → monitor, with monitoring feeding back into evaluation on change or drift.
- C. Approve → delete.
- D. Monitor only, after deployment.

### 14. Internal tools are not exempt

Why must *internal* tools go through the same lifecycle as third-party ones?

- A. They do not — internal tools are inherently safe.
- B. Internal tools take the same actions and drift the same way (upstream API/model changes silently regress quality), so they belong in the same registry and monitoring — just with a lighter evidence bar at Evaluate.
- C. Only because auditors require paperwork.
- D. To slow down the platform team deliberately.

### 15. Human accountability under autonomy

A fleet acts continuously and autonomously, with no human in most loops. What does this mean for human accountability?

- A. Autonomy removes human accountability.
- B. Autonomy *relocates* accountability: the architecture defines named human/role owners at consequential decision gates, a fleet RACI, intervention/override power, and a contestability path — accountability without the power to intervene is hollow.
- C. The model becomes the accountable party.
- D. Accountability is only needed if an incident occurs.

### 16. Lineage at fleet scale

A fleet runs several code and tool versions at once. Why must lineage record *which version* produced a given output?

- A. It does not matter; all versions behave identically.
- B. Without the version (agent/model/prompt/tool), "the agent did X" is unanswerable — you cannot enumerate a drifted or revoked component's blast radius or reconstruct the decision after the fact.
- C. Only to satisfy curiosity.
- D. Versions are random and not worth recording.

## Answer key

1. **B** — A control must resolve to a component, boundary, log, gate, or owner a reviewer can inspect. A stated commitment with no inspectable artifact is not yet a control (Ch. 1, the module throughline).
2. **B** — ISO/IEC 42001 is a certifiable AI *management system* standard (AIMS) on a Plan-Do-Check-Act cycle with an Annex A control set; it governs how the organization runs AI, not the model internals (Ch. 1).
3. **B** — The Act sets the obligation tier, RMF gives the risk process, 42001 gives the management system to run it repeatably and prove it; they have distinct, complementary jobs (Ch. 1).
4. **C** — Govern is cross-cutting: culture, roles, accountability, and risk tolerance wrapping Map/Measure/Manage (Ch. 1).
5. **B** — Risk tiers are domain-spanning; unrelated sectors can inherit the same structural high-risk obligations, giving the architect a common obligation set before sector-specific rules (Ch. 1).
6. **B** — Leading with a statute bakes one sector's assumptions into the architecture; leading horizontally yields a platform governable in general, parameterized per regime (Ch. 1–2).
7. **B** — Privacy, residency, auditability, accountability, retention — the five statute-independent primitives almost every regulation decomposes into (Ch. 2).
8. **B** — Privacy → classification at ingest + minimization at every boundary (including prompt, tools, logs) (Ch. 2).
9. **B** — An audit trail is tamper-evident, decision/action-focused on a fixed schema, retained per regime, and queryable — unlike ordinary logging (Ch. 2, Ch. 5).
10. **B** — Build the convergent core once, parameterize per regime, isolate structural deltas as pluggable components; never fork (Ch. 3).
11. **B** — Minor-consent is a structural divergence: a component built once and activated only where required (Ch. 3).
12. **B** — An agent's capabilities are its tools; each is an action/egress surface, so the registry is first-class governance (Ch. 4).
13. **B** — Evaluate → version → approve → monitor, with monitoring feeding back into evaluation (Ch. 4).
14. **B** — Internal tools take the same actions and drift the same way; same lifecycle, lighter evidence bar (Ch. 4).
15. **B** — Autonomy relocates (not removes) accountability: named gate owners, fleet RACI, intervention power, contestability (Ch. 5).
16. **B** — Lineage must record agent/model/prompt/tool versions so an output is traceable and a component's blast radius is enumerable (Ch. 5).
