# exercise-04: K-12 Edtech Agentic Constraints

**Estimated effort:** 3 hours

## Objective

Design the safety, human-in-the-loop, and hallucination-containment architecture for a student-facing agent serving minors. By the end you will have specified a tiered HITL model, a minor-safety control stack, and a hallucination-containment pipeline — the controls that make an agentic system safe to point at a child, beyond the privacy work of exercise-03.

## Background

This exercise covers material from:

- [Chapter 4 — Designing for Sensitive End-Users: K-12 Edtech](../04-sensitive-sectors-k12-edtech.md)
- [Chapter 2 — Regulated-Domain Constraints as Architecture](../02-regulated-domain-as-architecture.md) (accountability / HITL)

Privacy is necessary and insufficient. This exercise is about what the agent *says and does* to a child, and who is accountable for it.

## Scenario

The **TutorMesh tutoring agent** interacts directly with students (some under 13) and can: answer subject questions, generate practice problems, give feedback on submitted work, and draft a progress note that a teacher may forward to a parent. It has access to a curriculum-aligned knowledge base. You are designing the safety and output-governance architecture.

## Tasks

### 1. Bound the action space

- Define the agent's deliberately small tool set. Justify each tool, and list capabilities you **deny by design** (open web browsing, contacting a child outside the supervised channel, mutating a permanent record without a human).

### 2. Minor-safety control stack

- Specify input-side and output-side content safety calibrated for a child audience: age-inappropriate content, self-harm/crisis signals, grooming/exploitation, bullying.
- Design the **crisis-escalation path**: a self-harm signal must route to a named human, fast — not a canned refusal. Specify who and how.
- Specify interaction transparency: how the child and supervising adult know they are interacting with an AI, at an age-appropriate level.

### 3. Tiered human-in-the-loop

- Classify each of the agent's outputs by consequence and assign a HITL tier:
  - ephemeral hint / rephrasing,
  - generated practice problem,
  - feedback on submitted work,
  - draft progress note to a parent,
  - anything affecting a grade, record, IEP/504, or disciplinary outcome.
- For each high-stakes tier, specify the approval gate: what the educator sees (the claim, its sources, a confidence signal), how they approve/edit/reject, and how their identity and decision land in the audit log.
- Address the **rubber-stamp failure**: design the review surface so a teacher can realistically exercise the loop at classroom scale, or it becomes theater.

### 4. Hallucination containment

- Design the containment pipeline: curriculum-grounded retrieval for factual claims, citation + confidence on student-facing claims, in-domain constraint (a math agent defers on a history question), output-side claim-vs-source verification, and a bias toward "I don't know."
- Specify the gate: an ungrounded high-stakes factual assertion does **not** reach the student — it defers or escalates to the HITL gate.

### 5. The control specification

Produce a one-page spec table: each control, the risk it addresses, where it sits in the pipeline, its owner, and its evidence source.

## Starter guidance

Use this pipeline outline and control-spec table.

```text
# Student-facing safety + containment pipeline

  student input
      │
      ▼
  content-safety + age filter (in) ──▶ crisis signal? ──▶ escalate to named human
      │ ok
      ▼
  grounded retrieval (curriculum KB)        ← factual claims must trace to source
      │
      ▼
  output check: grounded? confident? in-domain?
      │                         │
   yes & low-stakes        no OR high-stakes
      │                         │
      ▼                         ▼
  to student (cited)       HITL gate: educator approves/edits/rejects;
                           identity + decision → audit log

# HITL tiering
| Output type            | Consequence | HITL tier            |
|------------------------|-------------|----------------------|
| ephemeral hint         | low         | post-hoc monitoring  |
| practice problem       | low-med     | sampled review       |
| feedback on work       | medium      | ...                  |
| progress note to parent| high        | gate before send     |
| affects grade/IEP/record| critical   | mandatory human decision |

# Control spec
| Control | Risk addressed | Pipeline location | Owner | Evidence |
|---------|----------------|-------------------|-------|----------|
| ...     | ...            | ...               | ...   | ...      |
```

## Acceptance criteria

You can demonstrate that:

- The agent's action space is deliberately small, with denied capabilities listed and justified.
- A minor-safety stack exists with **crisis escalation to a named human** (not a refusal), and age-appropriate interaction transparency.
- HITL is **tiered by consequence**: anything affecting a grade, record, IEP/504, discipline, or parent communication requires a human decision-maker, with identity and decision logged; the review surface is designed against rubber-stamping.
- A hallucination-containment pipeline grounds factual claims, cites + scores confidence, constrains to domain, gates ungrounded high-stakes assertions, and biases toward deferral.
- A control-spec table ties each control to a risk, a pipeline location, an owner, and an evidence source.

## Reflection

In `NOTES.md`:

1. Which output did you find hardest to tier, and where did you draw the line between post-hoc monitoring and a blocking gate?
2. How does your design avoid the rubber-stamp failure — what specifically makes the teacher's review fast enough to actually happen?
3. Give an example where biasing toward "I don't know" costs the student a correct answer. Why is that the right trade for a child-facing agent?

## Stretch goals

- Add an A/B-safe rollout: how would you test a new prompt version on this agent without exposing a child to an unreviewed regression? (Tie to mod-308 deployment safety.)
- Design the metric set a district would see: HITL approval rates, escalation counts, ungrounded-assertion catches, crisis-routing latency.
- Extend the containment design to a multilingual context where a translation plugin (exercise-02) sits between the agent and the student.
