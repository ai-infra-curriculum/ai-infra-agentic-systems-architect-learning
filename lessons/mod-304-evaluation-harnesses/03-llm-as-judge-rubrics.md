# Chapter 3 — LLM-as-Judge Rubrics

When the thing you need to score lives in free text — is this answer accurate? complete? properly cited? — no parser will do it, and human labeling won't scale to every CI run. The standard tool is **LLM-as-judge**: a model scores the output against a rubric you author. Used well, it is the workhorse of agent evaluation. Used carelessly, it is a random number generator that *feels* authoritative. The difference is rubric discipline and calibration.

Treat the judge as a **measurement instrument**. An instrument you haven't calibrated against a known standard produces numbers you can't trust. So the deliverable of this chapter is not "a judge prompt" — it's a judge you have *measured against human labels* and shown to agree well enough to gate on.

## Four axes worth scoring

For a tool-using, retrieval-grounded agent, four rubric axes carry most of the signal:

- **Accuracy** — are the claims factually correct and consistent with the provided evidence? (Not "fluent." Correct.)
- **Completeness** — does the answer address every part the task asked for? A correct answer to half the question is half an answer.
- **Citation faithfulness** — is every claim that needs support attributed to a source that *actually says it*? This catches the citation-hallucination failure: a real-looking citation pointing at a source that doesn't support the claim, or that the agent never retrieved.
- **Tool efficiency** — did the agent reach the answer without wasteful or redundant tool use? This is the axis that bridges to trajectory eval; score it from the trace, not the final answer.

Score each axis **separately**. A single "overall quality 1–10" score is uninterpretable — you can't tell a fluent-but-wrong answer from a correct-but-incomplete one, and you can't act on the number. Separate axes give you a vector you can threshold and diagnose independently.

## Anatomy of a gradeable rubric

A rubric the judge can apply consistently has four parts per axis: a **definition**, a **discrete scale with anchored levels**, **evidence the judge must cite**, and a **required output schema**.

**Use a short discrete scale, not a 1–10 continuum.** Models cannot meaningfully distinguish a 7 from an 8; they *can* distinguish "fully correct" from "minor error" from "major error." Three or four anchored levels per axis is the sweet spot. Anchor every level with a concrete description so two runs of the judge — and you and the judge — mean the same thing by each score.

```text
Axis: Citation faithfulness
  2 — Every claim requiring support cites a source, and each cited
      source actually supports the claim it's attached to.
  1 — Claims are cited, but at least one citation does not support
      its claim, OR a claim needing support is uncited.
  0 — Citations are absent, fabricated, or systematically mismatched.
  Evidence required: quote the specific claim+citation pair that set the score.
```

**Force the judge to cite evidence before scoring.** Requiring a one-line justification — a quoted claim, the mismatched citation — does two things: it makes the score auditable, and it measurably improves reliability, because the model commits to a reason before committing to a number. Put the reasoning field *before* the score in the output schema so the model generates it first.

```python
JUDGE_SYSTEM = """You are a strict evaluation judge. Score the agent's answer \
against the rubric. For each axis: first quote the specific evidence that \
determines the level, THEN assign the level. Use only the anchored levels. \
Do not reward fluency. If evidence is missing, score conservatively."""

RUBRIC = {
    "accuracy":     {2: "all claims correct & evidence-consistent",
                     1: "minor error not affecting the conclusion",
                     0: "a claim is wrong or contradicts the evidence"},
    "completeness": {2: "addresses every part of the task",
                     1: "addresses the main ask, misses a sub-part",
                     0: "leaves a primary part of the task unanswered"},
    "citation":     {2: "every supported claim cites a source that backs it",
                     1: "a citation is missing or doesn't support its claim",
                     0: "citations absent, fabricated, or mismatched"},
    "tool_efficiency": {2: "no redundant or unnecessary tool calls",
                        1: "one avoidable/redundant call",
                        0: "substantial wasted tool use"},
}

def build_judge_prompt(task, answer, evidence, trace) -> str:
    return (
        f"TASK:\n{task}\n\n"
        f"RETRIEVED EVIDENCE:\n{evidence}\n\n"
        f"AGENT ANSWER:\n{answer}\n\n"
        f"TOOL TRACE:\n{trace}\n\n"
        f"RUBRIC (use these exact levels):\n{RUBRIC}\n\n"
        "Return JSON: {axis: {evidence: str, level: int}} for every axis."
    )
```

**Pin the judge for reproducibility.** Use a fixed judge model and version, temperature 0, and a JSON-schema-constrained output. A judge whose model silently upgrades underneath you invalidates every historical comparison — version the judge exactly as you version the dataset.

## Calibration: prove the judge before you trust it

This is the step most teams skip, and it's the step that separates a real harness from theater. You **measure the judge against humans** on a gold set before gating on it.

1. **Build a small human-labeled gold set.** 30–80 cases, scored on the same axes and scale by one or more humans (ideally with a second labeler on a subset to measure human–human agreement — your ceiling). Cover the score range; an all-perfect gold set teaches you nothing about how the judge handles failures.
2. **Run the judge on the gold set** and compare per axis. For discrete scales, report **agreement** (exact-match rate) and, because some axes are ordinal, a rank/correlation measure (e.g., Cohen's kappa or Spearman). Don't average across axes — a judge can be reliable on accuracy and useless on tool efficiency.
3. **Inspect disagreements, fix the rubric, re-measure.** Most disagreements trace to an ambiguous level description, not a "dumb model." Tighten the anchor, add the failure to the rubric's examples, re-run. Iterate until per-axis agreement clears your bar — and approaches the human–human ceiling, which you cannot beat.

```python
def judge_agreement(human_labels, judge_labels, axis) -> dict:
    """Exact-agreement and mean-abs-error per axis on the gold set."""
    pairs = [(h[axis]["level"], j[axis]["level"])
             for h, j in zip(human_labels, judge_labels)]
    exact = sum(h == j for h, j in pairs) / len(pairs)
    mae = sum(abs(h - j) for h, j in pairs) / len(pairs)
    return {"axis": axis, "exact_agreement": exact, "mean_abs_error": mae,
            "n": len(pairs)}
```

If an axis can't reach acceptable agreement, that axis is not yet gateable — fall back to a deterministic proxy or human review for it, and keep iterating the rubric. Gating on an uncalibrated axis is worse than not gating: it blocks good releases and waves through bad ones at random.

## Bias-proofing the judge

LLM judges carry known, documented biases. Design them out:

- **Position / order bias** (in pairwise comparison) — judges favor the first response. If you compare candidate vs. reference, **swap order and average**, or score each absolutely against the rubric instead of comparing.
- **Verbosity / length bias** — judges reward longer answers. Counter it in the rubric ("length is not a positive; score completeness by coverage of the task, not word count") and watch for it in calibration.
- **Self-preference** — a judge tends to favor outputs from its own model family. Prefer a **different model** for the judge than the one under test where feasible, and note it as a risk where not.
- **Self-consistency** — run the judge **k times** on a sample and check variance. High variance means the rubric is underspecified; tighten anchors until repeated runs agree. For high-stakes axes, an **ensemble** (majority vote over a few judges/runs) buys stability at the cost of more calls.

The throughline: a judge's biases are systematic, so they are *designable-around*. Calibration is how you find them; rubric edits and ordering tricks are how you remove them.

## Where the judge fits in the harness

The judge is one scorer among several, not the whole harness. Deterministic checks ([Chapter 2](02-tool-call-correctness.md)) handle everything they can; the judge scores only the free-text axes that need it. Each case ends with a **score vector** — deterministic pass/fail plus per-axis judge levels — that the gating logic in [Chapter 4](04-eval-strategy-deployment-gate.md) thresholds into a release decision. Keep the judge's output structured and per-axis so that vector stays machine-readable end to end.

## Key takeaways

- Treat the judge as a **measurement instrument**: its value is only as good as its **calibration against human labels** — measure agreement per axis before you gate on it.
- Score **separate axes** (accuracy, completeness, citation faithfulness, tool efficiency), each on a **short anchored discrete scale**, never one blended 1–10.
- Force the judge to **quote evidence before scoring**, and pin the **model + version + temperature 0 + JSON schema** so scores are reproducible and auditable.
- **Calibrate** on a 30–80 case gold set, inspect disagreements, fix the rubric, re-measure; an axis that can't reach acceptable agreement is **not gateable** yet.
- Design out judge **bias**: swap order in pairwise comparisons, penalize verbosity in the rubric, prefer a different model than the one under test, and check self-consistency / ensemble for high-stakes axes.
