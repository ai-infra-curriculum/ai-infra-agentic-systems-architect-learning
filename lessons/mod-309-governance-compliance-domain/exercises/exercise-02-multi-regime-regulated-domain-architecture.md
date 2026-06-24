# exercise-02: Multi-Regime Regulated-Domain Architecture

**Estimated effort:** 3 hours

## Objective

Take **one** reference agent and produce the **control deltas** required to deploy it under four regulated regimes treated as **equal peers** — healthcare (HIPAA), finance (GLBA/SOX/PCI), public sector, and edtech (FERPA/COPPA) — then synthesize a **convergence/divergence comparison table** showing what you build once versus what you parameterize or build per regime. The deliverable proves that one architecture serves all four regimes without forking.

This is an **architecture** exercise: a per-regime delta analysis and a comparison table, not code. The core discipline is to keep the four sectors *balanced* — none is the default, and you must reason across at least all four.

## Background

This exercise covers material from:

- [Chapter 2 — Regulated Domain as Architecture](../02-regulated-domain-as-architecture.md) — the five primitives and the translation method.
- [Chapter 3 — Multi-Regime Mapping](../03-multi-regime-mapping.md) — convergence vs. divergence; the four peer regimes.

## The scenario

One reference agent, deployed (separately) into four sectors:

> A **case-assistance agent** that ingests records about an individual, retrieves relevant context, drafts recommendations, calls tools (read records, write a case note, notify a person, call an external verification API), and routes high-consequence actions to a human. The *same agent* is sold into a hospital network, a bank, a government agency, and a school district.

The data subject is, respectively, a **patient**, a **customer**, a **citizen**, and a **student (minor)** — but it is the *same agent*. Your job is to find the deltas, not rebuild it four times.

## Tasks

### 1. Decompose each regime into the five primitives

For **each** of the four regimes, fill a row using the primitives from [Chapter 2](../02-regulated-domain-as-architecture.md):

| Regime | Privacy (what's sensitive) | Residency | Auditability | Accountability (who decides) | Retention | Sector-specific extra |
| --- | --- | --- | --- | --- | --- | --- |
| Healthcare (HIPAA) | | | | | | |
| Finance (GLBA/SOX/PCI) | | | | | | |
| Public sector | | | | | | |
| Edtech (FERPA/COPPA) | | | | | | |

- Keep all four rows at the same level of rigor. Do not let any one regime dominate the analysis.

### 2. Identify the convergent core

- List the controls that are **the same mechanism across all four regimes**, differing only by parameter (per [Chapter 3](../03-multi-regime-mapping.md)). For each, state the shared mechanism and the per-regime parameter that varies (e.g., classification labels, retention schedule, gate authority).

### 3. Isolate the divergent deltas

- Separate divergence into **parameter-level** (configuration on the convergent core) and **structural** (a control one regime needs that others do not — a pluggable component).
- Name at least one structural delta per regime where one exists (e.g., verifiable parental consent for the minor in edtech; card-data scope reduction in finance; public-records disclosure in the public sector) and state which regimes activate it.

### 4. Build the convergence/divergence comparison table

Produce the synthesis table — this is the headline deliverable:

| Control | Healthcare | Finance | Public sector | Edtech | Verdict (converge param / converge / diverge-structural) |
| --- | --- | --- | --- | --- | --- |

- Cover at least: data classification, minimization, immutable audit log, accountability gate, residency, retention, plus the structural deltas you found.

### 5. Resolve stacking and conflict

- Show one case where two regimes **stack** on the same platform (e.g., the agent handles both health and payment data) and how per-class routing handles it.
- Give one case where regimes could **conflict** (e.g., a retention-minimum vs. a deletion-right on the same datum) and state how the architecture *surfaces and escalates* it rather than silently choosing.

## Starter guidance

The peer-regime frame to instantiate — keep them balanced:

```text
   ONE AGENT  ──▶  [ healthcare ] [ finance ] [ public ] [ edtech ]
                     same five primitives, four parameter profiles
   CONVERGE: classify · minimize · audit-log · gate · retention · access
   DIVERGE (param): region set · retention schedule · identifiable rule ·
                    gate authority
   DIVERGE (structural): minor-consent · card-data-scope · records-disclosure
```

A convergence-verdict shape to adopt:

```text
   control: immutable audit log
   healthcare: long retention  | finance: SOX integrity | public: public-records
   edtech: consent-bound retention
   verdict: CONVERGE — one tamper-evident log, retention as a parameter
```

You do **not** design the residency/retention controls in mechanism detail here (exercise-05) or the fleet-level specs (exercise-04) — reference them where a control points at them.

## Acceptance criteria

You can demonstrate that:

- All **four** regimes are decomposed into the five primitives at equal rigor, with no sector treated as the default or as a mere exception to another.
- The convergent core is identified, each with its shared mechanism and varying parameter.
- Divergence is correctly split into parameter-level vs. structural, with at least one structural delta named and attributed to the regimes that activate it.
- The convergence/divergence comparison table spans all four regimes and the listed controls, with a verdict per control.
- A stacking case and a conflict case are both handled — the conflict via surfacing/escalation, not a silent choice.

## Reflection

In `NOTES.md`:

1. Roughly what fraction of the total control set converged (build once) vs. diverged? What does that imply about the cost of adding a fifth regime?
2. Which regime tempted you to treat it as "the main one," and how did you keep the analysis balanced?
3. Pick one structural delta. Why can't it be reduced to a parameter on the convergent core — what makes it genuinely a new component?

## Stretch goals

- Add a fifth peer regime (e.g., a cross-border privacy regime like GDPR) and show how much of your convergent core absorbs it by parameter alone.
- Design the **regime-profile** object: the concrete set of parameters that configures the convergent core for one regime. What fields does it carry?
- Take the "most-restrictive-wins" resolution policy and show a case where it produces the wrong answer, requiring escalation instead of an automatic stricter default.
