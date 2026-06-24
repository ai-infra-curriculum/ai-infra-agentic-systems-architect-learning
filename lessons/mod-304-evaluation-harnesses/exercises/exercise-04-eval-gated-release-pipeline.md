# exercise-04: Wire an Eval-Gated Release Pipeline

**Estimated effort:** 3 hours

## Objective

Design and implement an **eval-gated release pipeline** that combines an offline regression gate with an online (shadow/canary) check, and prove it **blocks a deliberately regressing build** while **promoting a clean one**. By the end you'll have a mechanical, reproducible "should we ship?" decision — relative gates, hard gates, and coded promotion criteria — not a judgment call in a meeting.

## Background

This exercise covers material from:

- [Chapter 4 — Offline/Online Eval Strategy as a Deployment Gate](../04-eval-strategy-deployment-gate.md)

This exercise **composes the previous three**: the dual-lens scorer from [exercise-01](exercise-01-trajectory-eval-design.md), the tool-call harness from [exercise-02](exercise-02-tool-call-correctness-harness.md), and the calibrated judge from [exercise-03](exercise-03-llm-as-judge-rubric.md) are the scorers this gate runs.

## Prerequisites

- A versioned eval dataset (from exercise-01) and the scorers from exercises 01–03.
- Two builds of your agent: a **baseline** and a **candidate**. You will create a deliberately regressing candidate.
- Ability to replay a set of inputs as "shadow traffic" (a held-out input list works; no real production needed).

## Tasks

### 1. Produce a deployment-gate diagram

- Draw (as a text diagram in `pipeline.md`) the full path: change → offline gate → shadow eval → canary → full rollout, with the **rollback** arrows and the **label-flywheel** arrow back into the offline dataset.
- Label each transition with its **promotion criterion**.

### 2. Implement the offline gate

- Run all scorers over the **versioned dataset** for both baseline and candidate, **k runs per case**, and aggregate per axis.
- Implement two gate types:
  - **Relative gate** — block if any axis regresses more than ε versus baseline.
  - **Hard gate** — block (regardless of baseline) on any safety violation, unconfirmed destructive call, or schema-invalid rate above a tiny floor.
- Return a structured `GateResult` (pass/fail + per-axis diff + blocking reasons).

### 3. Implement a shadow/online check

- Replay your held-out "shadow" inputs through the candidate **without** surfacing output (no user impact).
- Score with the same harness using **reference-free** signals (judge-sampled axes, trajectory predicates, schema-validity rate, cost/latency per task).
- Block promotion on a hard-gate hit or a metric regression beyond threshold.

### 4. Implement a canary decision with rollback

- Simulate a canary: send a small slice of inputs to the candidate, the rest to baseline, compare on the same metrics over a **window** of N runs (not a single point).
- Implement **automated rollback**: any hard-gate hit or threshold regression reverts to baseline and routes the failing case **back into the offline dataset** (the flywheel).

### 5. Prove both directions

- **Regressing build:** create a candidate that introduces a regression (e.g., a prompt change that skips retrieval, or a tool change that emits an unconfirmed destructive call). Show the pipeline **blocks** it and names the gate that fired.
- **Clean build:** create a candidate with a genuine improvement (or no change) and show it **promotes** through all stages.

## Starter guidance

```python
from dataclasses import dataclass

@dataclass
class GateResult:
    passed: bool
    summary: dict          # per-axis candidate vs. baseline + deltas
    blocking: list[str]

def offline_gate(candidate, baseline, dataset, *, k=3, eps=0.02) -> GateResult:
    cand = run_suite(candidate, dataset, k=k)
    base = run_suite(baseline,  dataset, k=k)
    blocking, summary = [], {}
    for axis in cand:
        delta = cand[axis] - base[axis]
        summary[axis] = {"candidate": cand[axis], "baseline": base[axis],
                         "delta": delta}
        if delta < -eps:                                  # relative gate
            blocking.append(f"{axis} regressed {delta:+.3f}")
    if cand.get("safety_violations", 0) > 0:              # hard gate
        blocking.append("safety violation (hard gate)")
    return GateResult(not blocking, summary, blocking)

def promote(candidate, baseline, dataset, shadow_inputs):
    gate = offline_gate(candidate, baseline, dataset)
    if not gate.passed:
        return "BLOCKED: offline", gate.blocking
    if not shadow_ok(candidate, shadow_inputs):
        return "BLOCKED: shadow", None
    if not canary_ok(candidate, baseline, window=20):
        return "ROLLBACK: canary", None
    return "PROMOTED", gate.summary
```

## Acceptance criteria

You can demonstrate that:

- The offline gate compares candidate **vs. baseline** on a **versioned** dataset with **k runs per case**, and applies both **relative** and **hard** gates.
- A **deliberately regressing** candidate is **blocked**, and the output names which gate fired and why.
- A **clean** candidate **promotes** through offline → shadow → canary → full.
- Canary uses a **window** (not a single point), rollback is **automated**, and a rolled-back failure is **routed back into the offline dataset**.
- Promotion criteria are **coded/config**, not prose — the decision is reproducible.

## Reflection

In `NOTES.md`:

1. Which regression did your **relative** gate catch that an absolute threshold would have missed (or vice versa)? Why keep both?
2. What's the smallest dataset and k that gave you a *stable* gate decision (no random red/green)? How did you find it?
3. Walk through one rolled-back failure becoming a new offline case. Why does this make the same bug un-shippable twice?

## Stretch goals

- Wire the offline gate into a real CI step (GitHub Actions / pre-merge hook) that fails the build on a red gate.
- Add a **time-windowed** canary that requires stability across, say, 50 runs before promotion, and inject a slow-burn regression that only a window would catch.
- Integrate a tracing backend (LangSmith / Langfuse / Phoenix from [resources.md](../resources.md)) so the online stage scores real traces instead of replayed inputs.
