# exercise-01: ISO/IEC 42001 Controls Mapping

**Estimated effort:** 3 hours

## Objective

Produce a controls-mapping artifact that takes a concrete agentic platform and maps the relevant ISO/IEC 42001 clauses and Annex A controls onto its architecture — each requirement resolved to a real component, boundary, gate, or owner, with a named evidence source. By the end you will have the load-bearing governance deliverable of this module: a table an auditor could read and a team could build against.

## Background

This exercise covers material from:

- [Chapter 1 — Governance Frameworks for Agentic Systems](../01-governance-frameworks.md)
- [Chapter 2 — Regulated-Domain Constraints as Architecture](../02-regulated-domain-as-architecture.md)

You are mapping a *management-system* standard onto a system. The skill is translation, not citation: a clause that does not change a component or produce evidence has not been mapped.

## Scenario

Adopt this platform as your system under governance (or substitute one you know, keeping comparable scope):

> **"TutorMesh"** — an agentic platform serving K-12 districts. A fleet of agents provides student tutoring, generates draft lesson plans for teachers, and summarizes student progress for parents. Agents read from a student information system, call a curriculum knowledge base, and (for the progress-summary agent) write notes back to a teacher dashboard. The platform supports third-party plugins (e.g., a math-solver, a translation connector). It runs on managed cloud inference.

## Tasks

### 1. Scope and context (Clause 4)

- Write the AIMS scope statement: which agents, data, and use cases are in scope, and which interested parties are affected (name minors and districts explicitly).

### 2. Risk-based selection

- Run a lightweight risk assessment: list the top 6–8 risks of this platform (draw from the NIST GenAI Profile risk categories) and rate each.
- From your risk assessment, decide which **Annex A control areas** are in scope. Produce a short statement-of-applicability: for each Annex A control area, **include (with reason) or exclude (with reason)**.

### 3. The controls-mapping table

Build the core deliverable — one row per mapped requirement, with these columns:

| Framework requirement (clause / Annex A area) | Architectural realization | Risk(s) it treats | Owner (role) | Evidence source |
| --- | --- | --- | --- | --- |

- Cover at minimum: Clause 6 (risk treatment), Clause 9 (performance evaluation / monitoring), Clause 10 (improvement), and Annex A areas for the AI system life cycle, data for AI systems, third-party relationships, and AI system impact assessment.
- Every row must name a real component in TutorMesh **and** a concrete evidence source. No "we are committed to…" rows.

### 4. The NIST crosswalk

- For three of your rows, add the corresponding NIST AI RMF function/subcategory (GOVERN / MAP / MEASURE / MANAGE). Show that operating with the RMF and certifying against 42001 can share controls.

### 5. Impact assessment stub

- Write a one-page AI system impact assessment for the **progress-summary agent** (the one that writes parent-facing output): affected individuals, potential harms, and the controls that mitigate them.

## Starter guidance

Use this controls-mapping template. Fill every cell; an empty evidence column means the control is aspirational.

```text
# ISO/IEC 42001 Controls Mapping — <platform name>
# AIMS scope: <one paragraph>
# Statement of applicability: <Annex A area> = INCLUDED (reason) | EXCLUDED (reason)

| Framework ref           | Architectural realization        | Risk treated | Owner (role)       | Evidence source              |
|-------------------------|----------------------------------|--------------|--------------------|------------------------------|
| Cl. 6 risk treatment    | <component + control>            | R-..         | <role>             | <log / record / config>      |
| Annex A — life cycle    | <gated pipeline + eval>          | R-..         | <role>             | <pipeline def + eval results>|
| Annex A — third party   | <plugin registry + signing>      | R-..         | <role>             | <approval + signature logs>  |
| Annex A — data for AI   | <scoping + lineage tagging>      | R-..         | <role>             | <lineage records>            |
| Cl. 9 perf. evaluation  | <eval harness + traces>          | R-..         | <role>             | <dashboards + eval history>  |
| ...                     | ...                              | ...          | ...                | ...                          |
```

> Verify clause numbers and Annex A control areas against the standard itself ([resources.md](../resources.md)). Stating a clause wrong is worse than omitting it.

## Acceptance criteria

You can demonstrate that:

- A scoped AIMS statement and a risk-based statement-of-applicability exist, with **reasoned inclusions and exclusions** (not "include everything").
- The controls-mapping table has **at least 10 rows**, each naming a concrete TutorMesh component and a concrete evidence source.
- At least three rows carry a correct NIST AI RMF crosswalk.
- The progress-summary impact assessment names affected minors/parents and ties each harm to a mitigating control in the table.
- No row contains a vague commitment with no inspectable artifact behind it.

## Reflection

In `NOTES.md`:

1. Which requirement was hardest to translate into a *component* rather than a *policy*? What did you land on?
2. Name one Annex A area you excluded and defend the exclusion as you would to an auditor.
3. Where did 42001 and the NIST RMF give you the *same* control under different names?

## Stretch goals

- Add a per-tenant statement of applicability: which controls a specific district's deployment relies on.
- Identify two controls that are **over-governance** for a Tier 1 read-only agent in this platform and propose a proportionate alternative.
- Draft the management-review agenda (Clause 9.3) that would keep this mapping current quarterly.
