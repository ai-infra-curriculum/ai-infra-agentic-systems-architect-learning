# exercise-02: Quality and Drift Signals

**Estimated effort:** 3 hours

## Objective

Define the non-APM signal taxonomy for an agent and wire two of its signals — **faithfulness** and **quality drift** — into your traces. You'll first design the signal taxonomy as an artifact (which signals, what each measures, how it's computed, where it attaches), then implement asynchronous faithfulness scoring joined back to traces by `trace_id`, and finally detect a quality regression across two simulated deploys. By the end you'll have built the quality plane that APM cannot produce.

## Background

This exercise covers material from:

- [Chapter 2 — Agent Observability vs. Traditional APM](../02-observability-vs-apm.md)
- [Chapter 3 — Tracing Non-Deterministic, Long-Running Agents](../03-tracing-nondeterministic-agents.md) (deploy-segmented comparison)

Build on the instrumented agent from [exercise-01](exercise-01-otel-genai-span-design.md). For the faithfulness judge, reuse an LLM-as-judge rubric from [mod-304](../../mod-304-evaluation-harnesses/README.md). A RAG-style agent (retrieve → answer) makes faithfulness concrete; if yours isn't RAG, attach a simpler correctness rubric instead and keep the same wiring.

## Prerequisites

- The instrumented orchestrator-worker (or RAG) agent from exercise-01, emitting traces with stable `trace_id`s and `service.version`.
- An LLM-judge you can call (the rubric harness from mod-304, or a direct provider call).
- A store you can query traces from — your OTLP backend's API, or a local export to JSON.

## Tasks

### 1. The signal taxonomy artifact

Write `SIGNALS.md`. Define **at least four** non-APM signals (start from faithfulness, output quality, safety, task success). For each, specify:

- **What it measures**, in one sentence a non-expert understands.
- **How it's computed** — deterministic check, LLM-judge, or guardrail outcome — and on what input (the output? the output *and* retrieved context?).
- **Where it attaches** — span attribute, trace-level attribute, or a separate score joined by `trace_id`.
- **Sync or async** — inline (blocks the run) or post-hoc on a sample.

Explicitly note, for each, why APM's golden signals can't produce it.

### 2. Faithfulness scoring, joined by trace_id

- After a run completes, take its output **and** the retrieved context captured on the trace, and score faithfulness with the LLM-judge (claim supported by context: yes/no/partial → a 0–1 score).
- Run this **asynchronously** — not inline in the agent loop — and **join the score back to the trace by `trace_id`** (as a backend score/feedback, or a second span/event linked to the trace).
- Demonstrate a **faithful** run scoring high and a deliberately **unfaithful** run (inject a claim absent from the context) scoring low.

### 3. Quality drift across deploys

- Run a batch of N≥20 inputs under `service.version = A`. Record the faithfulness (or quality) distribution.
- Change *one thing* that should degrade quality (a worse prompt, a smaller model, a broken retrieval step) and tag it `service.version = B`. Run the same batch.
- Produce a **distribution comparison** grouped by `service.version`: means and a simple visual. Show the regression is attributable to the deploy, not run-to-run noise.

### 4. The drift alert rule

- Write the rule (in prose or code) that would have fired: e.g. "alert when 7-day mean faithfulness for the current `service.version` drops more than 0.05 below the prior version's mean." State your threshold and why it's neither too jumpy nor too numb.

## Starter guidance

```python
async def score_faithfulness(trace_id: str, output: str, context: list[str]) -> float:
    """LLM-judge: fraction of output claims supported by retrieved context. 0..1."""
    verdict = await judge(
        rubric="faithfulness",
        answer=output,
        evidence="\n".join(context),
    )
    return verdict.score  # 0..1

async def attach_score(backend, trace_id: str, name: str, value: float) -> None:
    """Join an async eval result back to its trace."""
    await backend.create_score(trace_id=trace_id, name=name, value=value)

# offline drift comparison
def compare_by_version(scores: dict[str, list[float]]) -> None:
    for version, vals in scores.items():
        mean = sum(vals) / len(vals)
        print(f"{version}: mean={mean:.3f}  n={len(vals)}")
```

You do **not** need to evaluate platforms (exercise-03) here — one backend is enough.

## Acceptance criteria

You can demonstrate that:

- `SIGNALS.md` defines ≥4 non-APM signals with measure / computation / attachment / sync-vs-async, and states why APM can't produce each.
- Faithfulness is scored **asynchronously** against the run's output *and retrieved context*, and the score is **joined back to the trace by `trace_id`** in your backend.
- A faithful run scores high and an injected-hallucination run scores low — the signal actually discriminates.
- A side-by-side distribution grouped by `service.version` shows the seeded regression as an attributable step-change, not noise.
- A drift alert rule is written with a justified threshold.

## Reflection

In `NOTES.md`:

1. How much extra latency/cost would inline faithfulness scoring have added, and why does async-on-a-sample win for production?
2. Your unfaithful run — did the judge catch *every* injected claim, or did some slip through? What does that say about trusting a single judge?
3. If the deploy B regression had been 0.02 instead of 0.10, would your alert have fired? Would it *should* have?

## Stretch goals

- Add a **safety** signal: a regex/classifier PII check on the output, recorded as a trace attribute, and show a leaking run flagged.
- Segment the drift comparison by `gen_ai.request.model` as well as `service.version` and disentangle a model change from a prompt change.
- Implement **tail-biased sampling** for which traces get the (expensive) faithfulness judge: 100% of errors and low-quality runs, a small fraction of the rest.
