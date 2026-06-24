# exercise-03: Author an LLM-as-Judge Rubric

**Estimated effort:** 3 hours

## Objective

Author a four-axis LLM-as-judge rubric (accuracy, completeness, citation faithfulness, tool efficiency), then **calibrate it against a human-labeled gold set** and prove per-axis agreement before declaring any axis gateable. By the end you'll treat the judge as a measurement instrument — measured, not assumed — and know exactly which axes you can trust to gate a release.

## Background

This exercise covers material from:

- [Chapter 3 — LLM-as-Judge Rubrics](../03-llm-as-judge-rubrics.md)

Use outputs from a retrieval-grounded, tool-using agent (yours from earlier exercises, or a fixed set of recorded answers + traces + retrieved evidence). You are scoring free-text answers, so you need real or recorded outputs across a *range* of quality.

## Prerequisites

- 30–80 agent outputs, each with: the task, the agent's answer, the retrieved evidence, and the tool trace.
- An LLM client capable of temperature 0 with JSON-schema-constrained output.
- The ability to label a gold set by hand (you, ideally plus one more labeler on a subset).

## Tasks

### 1. Write the rubric

- For each of the four axes — **accuracy, completeness, citation faithfulness, tool efficiency** — write a definition and a **short discrete scale** (3–4 anchored levels, e.g., 0/1/2). Anchor every level with a concrete description.
- Require the judge to **quote evidence before assigning each level**, and put the evidence field *before* the score in the output schema.
- Pin the judge: fixed model + version, temperature 0, JSON-schema output. Record the judge version string.

### 2. Build the gold set

- Hand-label **30–80 cases** on the same axes and scale. **Cover the score range** — include clear failures, not just good answers.
- Have a second labeler score a subset (≥10 cases) and compute human–human agreement per axis. This is your **ceiling**; the judge cannot beat it.

### 3. Measure agreement

- Run the judge on the gold set. Compute, **per axis**: exact-agreement rate and mean absolute error (and a rank measure like Spearman or Cohen's kappa for ordinal axes).
- Do **not** average across axes — report each separately.

### 4. Inspect, fix, re-measure

- Read the disagreements. For each, decide: ambiguous rubric anchor, or genuine judge error?
- Tighten anchors / add examples for the ambiguous ones; re-run; show agreement improving toward the human ceiling.

### 5. Declare gateability per axis

- Set a per-axis agreement bar (justify it relative to your human–human ceiling).
- Label each axis **gateable** or **not-yet-gateable**. For any not-yet-gateable axis, state your fallback (deterministic proxy or human review).

## Starter guidance

```python
JUDGE_SYSTEM = (
    "You are a strict evaluation judge. For each axis: first quote the "
    "specific evidence that determines the level, THEN assign a level from "
    "the anchored scale. Do not reward fluency or length. If evidence is "
    "missing, score conservatively."
)

RUBRIC = {
    "accuracy":        {2: "all claims correct & evidence-consistent",
                        1: "minor error, conclusion still holds",
                        0: "a claim is wrong or contradicts the evidence"},
    "completeness":    {2: "addresses every part of the task",
                        1: "main ask covered, a sub-part missed",
                        0: "a primary part left unanswered"},
    "citation":        {2: "every supported claim cites a source that backs it",
                        1: "a citation missing or not supporting its claim",
                        0: "citations absent, fabricated, or mismatched"},
    "tool_efficiency": {2: "no redundant/unnecessary tool calls",
                        1: "one avoidable/redundant call",
                        0: "substantial wasted tool use"},
}

def agreement(human, judge, axis):
    pairs = [(h[axis]["level"], j[axis]["level"]) for h, j in zip(human, judge)]
    exact = sum(a == b for a, b in pairs) / len(pairs)
    mae = sum(abs(a - b) for a, b in pairs) / len(pairs)
    return {"axis": axis, "exact": exact, "mae": mae, "n": len(pairs)}
```

## Acceptance criteria

You can demonstrate that:

- The rubric scores **four separate axes**, each on a **short anchored scale**, with the judge **quoting evidence before the score**.
- The judge is **pinned** (model + version + temperature 0 + JSON schema) and reproducible across runs.
- You measured **per-axis agreement** against a range-covering human gold set, with a **human–human ceiling** for at least one axis.
- After at least one inspect-fix-remeasure cycle, you can name which axes are **gateable** and what the fallback is for any that are not.

## Reflection

In `NOTES.md`:

1. Which axis had the lowest judge–human agreement, and what rubric edit improved it most?
2. Did you find a **bias** (verbosity, position, self-preference)? How did you detect it and what did you do?
3. For your worst axis, is the gap a judge limitation or a rubric-ambiguity limitation? How can you tell them apart?

## Stretch goals

- Run the judge **k=5 times** on 10 cases at temperature 0 and report self-consistency; if it varies, tighten the anchors until it stabilizes.
- Swap the judge to a **different model family** than the agent under test and re-measure self-preference effects.
- Implement a **pairwise** variant (candidate vs. reference) and show the order-swap-and-average debiasing in action.
