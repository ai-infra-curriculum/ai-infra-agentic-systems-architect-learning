# Chapter 4 — Designing for Sensitive End-Users: K-12 Edtech

Some sectors concentrate every governance pressure at once. K-12 education is the sharpest case in this module: the end-users are **minors**, the data is **student education records**, two distinct federal regimes apply (**FERPA** and **COPPA**), the output is **instructional content a child may believe**, and a wrong answer is not a bad UX moment — it can teach a falsehood or expose a child to harm. If your architecture can govern an agent that tutors a nine-year-old, it can govern most things. This chapter works that case end to end. The principles transfer to other sensitive sectors (pediatric health, juvenile services, accessibility populations); the specific statutes do not, so re-run Chapter 2's translation method for yours.

> **Scope note.** This chapter describes U.S. federal requirements at the architectural level; it is not legal advice. State student-privacy laws (e.g., California's SOPIPA) and contractual district terms add obligations. Engage counsel for the binding interpretation; your job is to make the architecture able to satisfy whatever counsel determines.

## FERPA — the student education record regime

The Family Educational Rights and Privacy Act (FERPA, 20 U.S.C. § 1232g; regulations at 34 CFR Part 99) governs **education records** held by schools and districts that receive U.S. Department of Education funding. The core rule: a school may not disclose **personally identifiable information (PII) from education records** without consent of the parent (or the "eligible student" once 18 or in postsecondary), with specific exceptions. The exception that matters for edtech vendors is the **"school official" exception** (34 CFR § 99.31(a)(1)): a vendor may receive education-record PII *without separate consent* when it:

- performs a service the school would otherwise use its own employees for;
- is under the **direct control** of the school regarding use and maintenance of the records;
- uses the records only for the **authorized purpose** and does not re-disclose them; and
- the school has specified the vendor as a school official with a legitimate educational interest in its annual notification.

Architectural consequences — each is a control, not a clause to admire:

- **Direct control → contractual + technical scoping.** "Direct control" lives in the contract/data-processing agreement *and* in the system: the vendor's agents access records only for the specified function, enforced by tool scopes and purpose tags (Chapter 2), not by promise.
- **No re-disclosure / no secondary use → boundary + tagged lineage.** Education-record data tagged "instructional" cannot flow to model-training, marketing, analytics-for-resale, or another student's context. The data-flow boundary from Chapter 2 enforces this; a path that lets student PII leave the instructional purpose is a FERPA violation waiting in your diagram.
- **Parental access/correction rights → per-subject indexing.** Parents have the right to inspect and request correction of the records; your stores (including agent memory and any vector index) must be queryable and correctable per student.
- **Recordkeeping of disclosures → audit log.** FERPA requires schools to keep records of certain disclosures; your audit trail (Chapter 2) is what makes the district able to honor that.

## COPPA — the under-13 regime

The Children's Online Privacy Protection Act (COPPA, 15 U.S.C. §§ 6501–6506; the FTC's COPPA Rule at 16 CFR Part 312) governs the online collection of **personal information from children under 13**. It applies to operators of online services directed to children, or who have actual knowledge they collect from under-13s. Its spine is **verifiable parental consent before collection**, plus data minimization, security, retention limits, and a parental right to review and delete.

The K-12 wrinkle: under FTC guidance, a **school can provide consent on the parents' behalf for COPPA purposes — but only when the data is collected and used solely for an educational, school-authorized purpose** (not for commercial purposes such as advertising). This is why the FERPA "instructional purpose only" boundary and the COPPA "no commercial use" boundary are *the same boundary in your architecture*. Architectural consequences:

- **Verifiable consent gate (school- or parent-provided) → fail-closed consent check.** No collection of a child's personal information proceeds without a recorded consent appropriate to the context. The consent check from Chapter 2 sits in the request path and fails closed.
- **No behavioral advertising / no commercial profiling → hard boundary.** The under-13 context forbids using the data to build a commercial profile or target ads. Architecturally: the regulated-data path has *no* connection to any ad/marketing/profiling subsystem — enforce it as an absent edge in the diagram, verified, not assumed.
- **Minimization and retention → scoping + lifecycle policy.** Collect only what the educational function needs; delete on schedule and on request. Retention here is a *ceiling* set by minimization, in tension with the audit-retention *floor* from Chapter 2 — reconcile by logging references/decisions rather than raw child PII.
- **Security → the full guardrails stack.** Reasonable security for children's data is not optional; it pulls in mod-306 controls wholesale.

> **FERPA and COPPA are not interchangeable.** FERPA is about *education records* and *disclosure*; COPPA is about *collection from under-13s* and *consent*. A K-12 agent typically must satisfy **both** at once, plus state law. Design to the union of their constraints.

## Safety for minors — beyond privacy

Privacy is necessary and insufficient. A student-facing agent must also be safe in what it *says and does*. Architect for:

- **Content safety calibrated to minors.** Input and output filtering (mod-306) tuned for a child audience — age-inappropriate content, self-harm and crisis signals, grooming/exploitation risks, bullying. Crisis-signal detection should route to a defined human escalation path, not a canned refusal.
- **Bounded action space.** A K-12 agent should have a deliberately small tool set. No open web browsing that can surface harmful content, no ability to contact a child outside the supervised channel, no actions that affect a permanent record without a human in the loop.
- **Interaction transparency.** The child (and the supervising adult) should be able to tell they are interacting with an AI, at an age-appropriate level.

## Human-in-the-loop on student-facing output

The defining control for this sector: **a human is accountable for what reaches the student**, proportionate to stakes.

- **Tier the HITL by consequence.** Low-stakes, ephemeral help (rephrasing a hint) may run with post-hoc monitoring. **Anything that affects a grade, a record, an IEP/504 plan, a disciplinary outcome, or a parent communication requires a human decision-maker in the path** — an approval gate (mod-308) before the action, with the educator's identity and decision captured in the audit log (Chapter 2). Solely-automated consequential decisions about a child are a line to design *against*.
- **The teacher is the loop, and the loop must be usable.** A HITL control a teacher cannot realistically exercise at classroom scale is theater. Design the review surface so the educator sees the agent's claim, its sources, and a confidence signal, and can approve/edit/reject quickly — or the loop gets rubber-stamped and you have automation with extra steps.
- **Escalation, not just approval.** Crisis content, repeated safety triggers, or low-confidence high-stakes output escalate to a named human, fast.

## Hallucination containment in instructional content

A child is an unusually credulous audience: confidently wrong instruction can implant a misconception that persists. Containment is architectural.

- **Ground, do not free-generate, on factual instruction.** Constrain factual claims to a vetted, curriculum-aligned knowledge source (retrieval-grounded, mod-303), rather than open-ended generation. The architecture should make it hard for the agent to assert a fact that is not in an approved source.
- **Cite and surface confidence.** Student-facing factual claims carry their source and a confidence signal — both so the supervising educator can verify (the HITL surface above) and so the child learns to check sources.
- **Constrain the claim space by subject.** A math-tutoring agent answering a history question outside its grounded domain should defer or escalate, not improvise.
- **Detect and gate, do not just hope.** Output-side checks (claim-vs-source verification, mod-306) flag ungrounded factual assertions before they reach the student; high-stakes assertions route to the HITL gate.
- **Fail toward "I don't know."** The safe default for a child-facing agent is admitting uncertainty and pointing to a human, not generating a plausible answer. Bias the system toward deferral on factual high-stakes content.

```text
  student question
        │
        ▼
  content-safety + age filter (in) ──┐ unsafe → escalate to named human
        │                            │
        ▼                            │
  grounded retrieval (curriculum) ───┘
        │  (factual claims must trace to approved source)
        ▼
  output check: grounded? confident? in-domain?
        │                    │
   yes, low stakes      no / high stakes
        │                    │
        ▼                    ▼
  to student          HITL gate (educator approves/edits/rejects,
  (cited)             decision + identity logged)
```

## Key takeaways

- K-12 edtech concentrates every governance pressure: minors, education records, **FERPA and COPPA simultaneously**, plus state law — design to the union of constraints, with counsel for binding interpretation.
- FERPA's **school-official exception** turns into contractual + technical direct-control scoping, a no-re-disclosure/no-secondary-use data boundary, per-student access/correction, and disclosure recordkeeping. COPPA's **verifiable parental consent** (school-provided only for solely-educational purposes) turns into a fail-closed consent gate and a hard "no commercial profiling" boundary — which is the *same* boundary FERPA's purpose limit draws.
- Safety for minors means content filtering tuned for children, a deliberately small action space, and crisis escalation to a named human — privacy alone is insufficient.
- **HITL on student-facing output, tiered by consequence** (a human decides anything touching grades, records, IEPs, discipline, or parent comms), is the defining control; pair it with **hallucination containment** — curriculum-grounded retrieval, cited claims, in-domain constraint, output gating, and a bias toward "I don't know."
