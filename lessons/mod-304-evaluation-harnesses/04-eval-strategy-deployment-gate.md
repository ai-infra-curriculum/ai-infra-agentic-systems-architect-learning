# Chapter 4 — Offline/Online Eval Strategy as a Deployment Gate

You now have the scorers: trajectory and final-state checks ([Chapter 1](01-trajectory-vs-final-state-eval.md)), layered tool-call correctness ([Chapter 2](02-tool-call-correctness.md)), and calibrated LLM-judge rubrics ([Chapter 3](03-llm-as-judge-rubrics.md)). This chapter is the architecture that turns those scorers into a **deployment gate** — a system that decides, automatically, whether a change is safe to ship, and that keeps watching after it ships.

The strategy has two halves. **Offline eval** runs *before* deploy, against a fixed dataset, in CI — it answers "did this change regress?" **Online eval** runs *after* deploy, against live (or shadow) traffic — it answers "is it actually working on the real distribution?" Neither is sufficient alone. Offline catches regressions but can't cover the long tail of real inputs; online sees reality but can't block a bad build before users hit it. The architect wires both into one promotion pipeline.

## Offline eval: the regression gate

Offline eval is the gate that runs on every change to the agent — prompt edits, tool changes, model swaps. It executes the agent over a **versioned dataset** and applies all scorers, producing a per-case score vector and an aggregate.

Its design constraints:

- **Deterministic where it can be.** Fix the dataset version, the judge model/version, and judge temperature 0. The agent under test may still be nondeterministic; control for it by running each case **k times** and reporting the distribution (e.g., pass rate, not a single pass/fail), so a flaky case doesn't randomly red/green the build.
- **Fast and parallel.** It's on the critical path of every merge. Run cases concurrently, short-circuit the judge behind deterministic checks (Chapter 2's cascade), and keep the dataset large enough to be representative but small enough to finish in minutes.
- **Compared against a baseline, not an absolute.** The gate question is "did this build regress relative to the current production build?" Score the candidate *and* the baseline on the same dataset version and diff. An absolute threshold ("must score >0.9") drifts; a **relative regression check** ("no axis dropped more than ε, no new hard failures") is what actually protects you.

```python
from dataclasses import dataclass

@dataclass
class GateResult:
    passed: bool
    summary: dict          # per-axis candidate vs. baseline
    blocking: list[str]    # human-readable reasons it failed

def offline_gate(candidate, baseline, dataset, *, k=3, eps=0.02) -> GateResult:
    cand = run_suite(candidate, dataset, k=k)   # per-axis aggregate scores
    base = run_suite(baseline,  dataset, k=k)
    blocking, summary = [], {}

    for axis in cand:
        delta = cand[axis] - base[axis]
        summary[axis] = {"candidate": cand[axis], "baseline": base[axis],
                         "delta": delta}
        if delta < -eps:                         # relative regression
            blocking.append(f"{axis} regressed {delta:+.3f} (> {eps} drop)")

    # Hard gates: absolute floors that must never be crossed regardless of baseline.
    if cand["safety_violations"] > 0:
        blocking.append("safety violation present (hard gate)")

    return GateResult(passed=not blocking, summary=summary, blocking=blocking)
```

Two gate *types* live here. **Relative gates** block regressions versus baseline (most axes). **Hard gates** are absolute floors that block regardless of baseline — any safety violation, any unconfirmed destructive tool call, schema-invalid output above a tiny rate. Hard gates never trade off against an improvement elsewhere; a build that's better on accuracy but introduces one destructive-tool safety violation does not ship.

## Online eval: watching the real distribution

Passing offline is necessary, not sufficient — the dataset is a fixed sample of a moving world. Online eval scores production behavior:

- **Online (shadow) eval before exposure.** Replay live traffic, or a sampled shadow of it, through the candidate without surfacing its output to users. Score with the same harness. This catches distribution shift the offline set missed *before* a real user is affected.
- **Canary / progressive rollout.** Route a small traffic slice (e.g., 5%) to the candidate, score it online, and compare against the baseline slice on the same metrics. Automated rollback triggers on a hard-gate violation or a metric regression beyond threshold.
- **Continuous online metrics.** Not every production request has a label, so lean on **reference-free** signals: judge-scored samples, trajectory predicates (retrieved-before-answering rate, tool-budget-exceeded rate, unconfirmed-destructive-call count), schema-validity rate, and cost/latency per task. Sample-and-judge a fraction continuously; alert on drift.
- **The label flywheel.** Production failures — especially online-flagged ones and human-overridden cases — are the highest-value new eval cases. Feed them back into the **offline** dataset (Chapter 1's "curate from real traces"). Today's online incident becomes tomorrow's offline regression test, so the same failure can never ship twice.

## The promotion pipeline

Put it together as one gated path from change to production:

```text
  change (prompt / tool / model)
        │
        ▼
  ┌───────────────────────────────────────────────┐
  │ OFFLINE GATE  (CI, versioned dataset)          │
  │   • deterministic checks (selection, schema,   │
  │     value, trajectory predicates)              │
  │   • calibrated LLM-judge rubric (per axis)     │
  │   • candidate vs. baseline diff, k runs/case   │
  │                                                │
  │   relative regression? ──┐   hard-gate hit? ──┐│
  └──────────┬───────────────┼──────────────────┬─┘│
             │ pass           │ FAIL → block     │  │
             ▼                ▼                  ▼  │
  ┌───────────────────────┐  ◀── fix & re-run ──────┘
  │ SHADOW / ONLINE EVAL  │
  │   replay live traffic │
  │   through candidate,  │
  │   score, no user impact│
  └──────────┬────────────┘
             │ pass
             ▼
  ┌───────────────────────┐      hard gate / metric
  │ CANARY  (5% traffic)  │──────  regression ──────▶ AUTO-ROLLBACK
  │   online metrics vs.  │
  │   baseline slice      │
  └──────────┬────────────┘
             │ stable over window
             ▼
  ┌───────────────────────┐
  │ FULL ROLLOUT          │
  │   candidate → baseline │
  │   continuous online    │──── flagged failures ──▶ back to OFFLINE dataset
  │   eval continues       │       (label flywheel)
  └───────────────────────┘
```

**Promotion criteria** — the explicit conditions for each transition — are the contract that makes this auditable:

- **Merge → offline gate:** every change runs the suite; no merge to the deploy branch without a green relative gate and zero hard-gate hits.
- **Offline → shadow:** offline green; shadow eval on replayed traffic shows no hard-gate violation and no metric regression beyond threshold.
- **Shadow → canary:** shadow clean over a minimum sample size.
- **Canary → full:** canary metrics within threshold of baseline over a **time window** (not a single point — wait out the noise), zero hard-gate hits, automated rollback armed throughout.
- **Any stage → rollback:** a single hard-gate violation, or a metric regression past threshold, reverts to baseline automatically and routes the failure back into the offline dataset.

Write these as code/config, not tribal knowledge. The whole value of an eval gate is that the *decision* is mechanical and reproducible — the moment "should we ship?" becomes a judgment call in a meeting, you've lost the property that makes the harness an architectural control.

## What this buys the architecture

An eval-gated pipeline converts agent quality from a vibe into a controlled variable. You can change the model, rewrite a prompt, or add a tool and *know* — within the coverage of your dataset and the calibration of your judge — whether the system got better or worse, before users do. Its limits are honest: it's only as good as the dataset's coverage, the judge's calibration, and the hard gates you thought to write. Those limits are why the label flywheel matters — the gate gets stronger every time production teaches it a new failure.

## Key takeaways

- **Offline eval** (CI, versioned dataset, before deploy) catches regressions; **online eval** (shadow + canary + continuous, after deploy) catches the real distribution. You need both — neither covers the other's blind spot.
- Gate **relative to a baseline** on most axes (no axis drops > ε) and apply **absolute hard gates** (safety, destructive-tool, schema-validity) that block regardless of improvements elsewhere.
- Control agent nondeterminism by running **k times per case** and gating on the distribution, not a single pass/fail.
- Roll out progressively — **shadow → canary → full** — with **explicit, coded promotion criteria** and **automated rollback** armed throughout; canary must hold over a *time window*, not a single point.
- Run a **label flywheel**: production/online failures become new offline cases, so a shipped bug becomes a permanent regression test and can't ship twice.
