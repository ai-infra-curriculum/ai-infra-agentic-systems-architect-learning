# exercise-02: Orchestration Pattern Catalog

**Estimated effort:** 3 hours

## Objective

Build your own annotated **catalog of the five orchestration patterns**, then map a set of real tasks onto them — including tasks that need **composed** patterns (a router on the outside, a chain inside one branch). The deliverable is a reference document you can reuse on real designs: each pattern with its shape, fit, cost, failure mode, and at least one worked task mapping.

## Background

This exercise covers material from:

- [Chapter 2 — The Orchestration Pattern Catalog](../02-orchestration-patterns.md)
- [Chapter 1 — Workflows vs. Agents: The Foundational Choice](../01-workflows-vs-agents.md)

The anchor reference is Anthropic's [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents), which defines all five building blocks.

## Prerequisites

- You can name the five patterns and which carry model-driven control flow.
- Comfort drawing simple architecture diagrams (ASCII or a diagramming tool).

## Tasks

### 1. Catalog each pattern

For **prompt chaining, routing, parallelization, orchestrator-workers, and evaluator-optimizer**, write a one-page entry containing: a diagram, the task shape it fits, its relative cost, its predictability, its primary failure mode, and one explicit "do not use it for…" anti-example.

### 2. Map ten tasks to patterns

Assign each of these to a pattern (or a composition), and justify in one or two sentences:

1. Translate a document, then summarize it, then format it as a brief.
2. Route inbound support tickets to refund / technical / billing handlers.
3. Review a code change for security, style, and performance simultaneously.
4. Answer an open-ended research question by deciding which sources to pull.
5. Generate a marketing tagline, critique it against brand guidelines, and revise until it passes.
6. Classify an email's language, then run the language-specific responder.
7. Make a coordinated edit across an unknown set of files in a repository.
8. Grade a free-text answer three times and take the majority safety verdict.
9. Draft a contract clause, then check it for prohibited terms before release.
10. Personalize an onboarding sequence from a fixed CRM schema.

### 3. Design two composed systems

- Pick two tasks above that need more than one pattern. Draw the composed architecture (e.g., outer routing → one branch is a chain → another branch is orchestrator-workers).
- Label, on each diagram, exactly where control flow stops being fixed-path and becomes model-driven (the workflow/agent boundary from Chapter 1).

### 4. Tabulate the tradeoffs

- Produce a single comparison table across all five patterns: control flow (fixed vs. model-driven), relative cost, predictability, and the one bounding control each needs (e.g., orchestrator-workers needs a fan-out cap; evaluator-optimizer needs an iteration cap).

## Starter guidance

Use this catalog-entry template per pattern, and the mapping template per task. These are documentation structures, not code skeletons — the work is the analysis, not the syntax.

```text
PATTERN ENTRY

  Name:            <pattern>
  Diagram:         <ASCII sketch of the control flow>
  Fits:            <task shape>
  Relative cost:   <low | medium | high | variable>
  Predictability:  <high | medium>
  Bounding control:<the cap/gate it needs to stay safe>
  Primary failure: <how it goes wrong>
  Do NOT use for:  <anti-example>
```

```text
TASK MAPPING

  Task:        <task>
  Pattern(s):  <pattern, or composition A -> B>
  Boundary:    <where control flow turns model-driven, if anywhere>
  Rationale:   <1-2 sentences tied to the task shape>
```

## Acceptance criteria

You can demonstrate that:

- All five patterns have a complete catalog entry including a diagram and a bounding control.
- All ten tasks are mapped, each with a one-or-two-sentence rationale tied to task shape (not "it seemed agentic").
- At least two tasks are correctly identified as **compositions** and drawn as such.
- The composed diagrams label the fixed-path → model-driven boundary explicitly.
- The tradeoff table is consistent with Chapter 2 (chaining/routing/parallelization are predictable; orchestrator-workers and evaluator-optimizer carry the variance).

## Reflection

In `NOTES.md`:

1. Which task was most tempting to over-pattern (you wanted orchestrator-workers but a chain or routing was enough)? What pulled you toward the heavier option?
2. For each pattern, name the single bounding control without which it becomes a cost or reliability incident.
3. Where in your two composed systems is the highest-risk boundary, and why?

## Stretch goals

- Add a sixth entry for the **autonomous agent loop** (no fixed control flow at all) and position it relative to orchestrator-workers — what distinguishes "the orchestrator decides subtasks" from "the agent decides everything"?
- For three of your task mappings, write the **downgrade**: the simpler pattern you would fall back to if cost became the dominant constraint, and what you would lose.
- Take one composed system and annotate it with where you would place evals and observability spans (forward reference to mod-304 and mod-305).
