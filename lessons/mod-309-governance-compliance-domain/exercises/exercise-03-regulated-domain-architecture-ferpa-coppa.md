# exercise-03: Regulated-Domain Architecture (FERPA/COPPA)

**Estimated effort:** 3 hours

## Objective

Translate two real federal regulations — FERPA and COPPA — into a concrete agentic architecture: data-flow boundaries, immutable audit logging, and named accountability. By the end you will have run the Chapter 2 translation method on actual statutes and produced a data-flow diagram and a compliance matrix where every regulatory requirement resolves to a boundary, a control, and an evidence source.

## Background

This exercise covers material from:

- [Chapter 2 — Regulated-Domain Constraints as Architecture](../02-regulated-domain-as-architecture.md)
- [Chapter 4 — Designing for Sensitive End-Users: K-12 Edtech](../04-sensitive-sectors-k12-edtech.md) (FERPA/COPPA specifics)

This is architecture translation, not legal analysis. You are not deciding what the law means; you are designing a system able to satisfy it. Cite the primary sources from [resources.md](../resources.md) — the clause number is the evidence.

> **Not legal advice.** Treat statutory text at the architectural level; binding interpretation is counsel's job. State law (e.g., SOPIPA) and district contracts add obligations beyond this exercise.

## Scenario

Design the regulated-data architecture for the **TutorMesh student-tutoring agent**: a child under 13 interacts with an agent that reads the student's education records to personalize tutoring and produces instructional output. The vendor operates as a "school official" under the district's control. Managed cloud inference is used.

## Tasks

### 1. Run the translation method

For each requirement below, fill the five questions (data / flow / action / control / evidence) from Chapter 2:

- **FERPA — school-official exception** (34 CFR § 99.31(a)(1)): records used only for the authorized function, under district direct control, no re-disclosure.
- **FERPA — purpose / no secondary use**: education-record data must not flow to model training, marketing, analytics-for-resale, or another student's context.
- **FERPA — parental access/correction**: parents can inspect and request correction of records.
- **COPPA — verifiable consent** (16 CFR Part 312): no collection of a child's personal information without appropriate consent (school-provided only for solely-educational purposes).
- **COPPA — no commercial profiling / behavioral advertising** for under-13s.
- **COPPA — minimization and retention limits**.

### 2. Draw the data-flow diagram

- Produce a data-flow diagram (as `text`) from the student information system through scoping/redaction, the consent/purpose check, the agent context, tool calls, and the audit log.
- Mark every **boundary** the regulations care about. Critically, show the *absent* edge: there is **no path** from the regulated-data flow to any marketing/profiling/training subsystem. An absent edge you verify is a control.

### 3. Design the audit architecture

- Specify the audit log: what each entry captures, how it is made tamper-evident (append-only / hash-chained / WORM), how long it is retained, and how it is queried ("show every record access for student X in window Y").
- Reconcile the **tension** between COPPA minimization/retention *ceilings* and audit-retention *floors* (hint: log references and decisions, not raw child PII).

### 4. Design accountability

- Name the accountable owner for the tutoring agent and write the ADR that records the decision "this agent may read student records autonomously but may not disclose externally."
- Identify which actions require a human-in-the-loop gate and how the human's decision lands in the same audit record.

### 5. The compliance matrix

Produce the deliverable: one row per requirement.

| Requirement (cite source) | Data class | Boundary / control | Evidence source | Owner |
| --- | --- | --- | --- | --- |

## Starter guidance

Use this translation-and-matrix template.

```text
# Translation method (per requirement)
Requirement: <FERPA/COPPA cite>
  1. WHAT DATA?    <class>
  2. WHAT FLOW?    <entry → rest → exit>
  3. WHAT ACTION?  <agent capability that touches it>
  4. WHAT CONTROL? <boundary / scope / consent gate / redaction / log>
  5. WHAT EVIDENCE?<log entry / approval record / DPA clause>

# Data-flow diagram (mark boundaries; show the ABSENT edge to marketing/training)

  SIS ──▶ scoping+redaction ──▶ [consent/purpose check: fail closed] ──▶
  agent context ──▶ scoped tools ──▶ audit log
      (no edge: agent context ──X──▶ marketing / profiling / training)

# Compliance matrix
| Requirement (cite) | Data class | Boundary/control | Evidence | Owner |
|--------------------|-----------|------------------|----------|-------|
| FERPA §99.31(a)(1) | edu record| scope + purpose tag + DPA | access log + DPA | <role> |
| COPPA 16 CFR 312   | child PII | fail-closed consent gate  | consent record | <role> |
| ...                | ...       | ...                       | ...      | ...    |
```

## Acceptance criteria

You can demonstrate that:

- Every listed requirement has a completed five-question translation ending in a **control and an evidence source** (not a policy statement).
- The data-flow diagram marks the regulated boundaries **and** shows the verified absence of any path to marketing/profiling/training.
- The audit architecture is tamper-evident and queryable by subject, and the minimization-vs-retention tension is reconciled explicitly.
- A named accountable owner and an ADR exist; consequential actions route through a HITL gate captured in the audit record.
- The compliance matrix cites the primary source for each row (correct statute/regulation reference) and names a control, evidence, and owner per row.

## Reflection

In `NOTES.md`:

1. FERPA's purpose-limitation boundary and COPPA's no-commercial-use boundary turned out to be the *same* boundary in your design. Explain why, in architectural terms.
2. Where does "agent memory" or a vector index become a regulated data store you had to include in deletion and access paths?
3. Which requirement was *only* satisfiable contractually (in the DPA) versus technically — and how does your architecture make the contractual promise enforceable?

## Stretch goals

- Add the parental data-subject-rights flow end to end: a parent requests deletion; trace it through every store including agent memory and the vector index.
- Region-pin the inference and storage and show how a managed cloud model API is treated as a sub-processor under the DPA.
- Add a second jurisdiction (e.g., a California district under SOPIPA) and note where the union of constraints tightens your boundaries.
