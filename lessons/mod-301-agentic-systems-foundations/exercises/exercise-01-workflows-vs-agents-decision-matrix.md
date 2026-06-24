# exercise-01: Workflows vs. Agents Decision Matrix

**Estimated effort:** 2 hours

## Objective

Build a **reusable decision matrix** that, given a candidate system, tells you whether to architect it as a workflow or an agent — and produces a defensible written rationale, not a gut call. You will apply it to five candidate systems of deliberately mixed difficulty, including at least one trap where the "exciting" answer (agent) is the wrong one. The deliverable is a decision instrument you could hand to a teammate and a set of filled-in judgments.

## Background

This exercise covers material from:

- [Chapter 1 — Workflows vs. Agents: The Foundational Choice](../01-workflows-vs-agents.md)
- [Chapter 4 — The Discipline of Starting Simple](../04-start-simple-discipline.md)

The anchor reference is Anthropic's [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents). Re-read its workflow/agent distinction before starting.

## Prerequisites

- You can read an agent loop and a tool call and recognize who owns control flow.
- Comfort writing structured decision rationale (you are producing a document, not code).

## Tasks

### 1. Design the matrix

- Define **6–8 weighted criteria** that discriminate workflow from agent (start from Chapter 1's table: step enumerability, path shape stability, cost/latency predictability requirement, deterministic-replay requirement, need for mid-task course-correction, input-space boundedness). Add any you can defend.
- Assign each criterion a **weight** reflecting how strongly it should move the decision, and define a scoring scale (e.g., −2 = strongly workflow … +2 = strongly agent).
- Define the **decision rule**: how the weighted score maps to a recommendation, and where the "bias toward workflow" tie-break lives.

### 2. Score five candidate systems

Apply the matrix to all five:

1. A weekly compliance report generated from three fixed data sources.
2. A customer-support assistant that resolves account issues by investigating across multiple systems.
3. A code-review bot that checks each PR for security, style, and performance.
4. An onboarding email sequence personalized from a CRM record.
5. An "AI research analyst" that answers open-ended questions by deciding which sources to consult.

### 3. Produce the recommendation per candidate

- For each, record the per-criterion scores, the weighted total, and the resulting recommendation (workflow / agent / hybrid).
- Where you recommend **hybrid** (a workflow that escalates to an agent for a hard fraction), state the escalation trigger explicitly.

### 4. Stress-test the matrix

- Identify the candidate where your matrix's output felt **least confident** and explain why.
- Construct one **adversarial candidate** that you believe should be a workflow but that a naive reader would call an agent (and vice versa). Run it through the matrix and confirm the matrix gets it right; if it does not, fix the weights.

## Starter guidance

Use this matrix template as your decision instrument. Fill one table per candidate; the weights stay constant across candidates (that is what makes it a *reusable* instrument).

```text
DECISION MATRIX — <candidate name>

  Criterion                              Weight   Score(-2..+2)   Weighted
  -------------------------------------  ------   -------------   --------
  Steps enumerable in advance?            3        ...             ...
  Path shape stable across runs?          3        ...             ...
  Predictable cost/latency required?      2        ...             ...
  Deterministic replay required?          2        ...             ...
  Needs mid-task course-correction?       3        ...             ...
  Input space bounded & known?            2        ...             ...
  <your added criterion>                  ?        ...             ...
  -------------------------------------  ------   -------------   --------
  WEIGHTED TOTAL                                                   ...

  Decision rule:  total <= -4  -> workflow
                  -3 .. +3      -> hybrid (workflow w/ agent escalation)
                  total >= +4   -> agent
  Tie-break:      ambiguous -> workflow (bias toward predictability)

  RECOMMENDATION: <workflow | hybrid | agent>
  Escalation trigger (if hybrid): <when does it hand off to an agent?>
  One-line rationale: <why>
```

Tune the weights and thresholds to *your* defense of the patterns — the numbers above are a starting point, not a fixed answer.

## Acceptance criteria

You can demonstrate that:

- The matrix has weighted, scored criteria and an explicit decision rule with a stated tie-break.
- All five candidates are scored with per-criterion values and a recommendation each.
- At least one candidate is correctly identified as a **trap** (the naive answer differs from the matrix's answer), with the reasoning written out.
- Every recommendation has a one-line rationale tied to specific criteria, not vibes.
- The same weights are used across all candidates (the instrument is reusable, not retrofitted per case).

## Reflection

In `NOTES.md`:

1. Which single criterion did the most work in separating workflows from agents across your five candidates?
2. Where did the "bias toward workflow" tie-break change an outcome you would otherwise have called the other way?
3. For your hybrid recommendations, what would have to change about the input for you to collapse the hybrid into a pure workflow?

## Stretch goals

- Add a **cost-of-being-wrong** dimension: weight the matrix differently for high-stakes vs. low-stakes systems and show how that flips at least one recommendation.
- Turn the matrix into a one-page decision aid (flowchart + scoring rubric) suitable for a team wiki.
- Re-score candidate 2 and candidate 5 assuming a 100x increase in request volume; document which recommendations change and why (ties to the economics test in Chapter 4).
