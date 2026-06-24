# exercise-01: Design a Trajectory Eval

**Estimated effort:** 3 hours

## Objective

Design and implement an evaluation that scores a multi-step agent through **both** lenses — trajectory (the path) and final-state (the outcome) — and demonstrate that each lens catches a failure the other misses. By the end you'll have a versioned dataset, a dual-lens scorer, and a concrete example of the "outcome correct, trajectory unsafe" quadrant.

## Background

This exercise covers material from:

- [Chapter 1 — Trajectory vs. Final-State Evaluation](../01-trajectory-vs-final-state-eval.md)

Use any model provider and a small tool-using agent — ideally one you built in [mod-302](../mod-302-multi-agent-orchestration/README.md), or a stub agent whose traces you script. You do not need a production agent; you need *traces* (sequences of messages and tool calls) to score.

## Prerequisites

- An agent (or scripted traces) that emits a serializable trace: ordered `message` / `tool_call` / `tool_result` steps.
- A task domain with at least one retrieval tool and one action tool (e.g., a support agent: `lookup_order`, `issue_refund`, `send_email`).
- Python with a schema/validation library available.

## Tasks

### 1. Define the trace and case schema

- Define a serializable `Trajectory` (ordered steps with `kind` and, for tool calls, `tool` + `args`).
- Define a dataset `Case` carrying: `input`, a `final_state` reference, and a list of `trajectory_assertions` (predicates / required-step sets) — **not** a golden transcript.
- Commit the dataset as a versioned file (JSON/YAML) and record its version/commit hash.

### 2. Build a final-state scorer

- Compare the agent's end artifact / world state to the case's `final_state` reference. Use exact-match or a parser where the output is structured; use a single judge call only if the outcome is free text.
- Return a strict pass/fail per case.

### 3. Build a trajectory scorer

- Implement at least three predicate checks from the chapter: `required_steps_present` (order-allowing superset), `retrieved_before_answering`, and a budget check (`tool_budget_respected(max=N)`).
- Add one safety predicate for your domain, e.g., `no_unconfirmed_destructive_call` (no `issue_refund` / `delete_*` without a preceding confirmation step).
- Return per-predicate results, not a single blended boolean.

### 4. Curate cases that separate the lenses

- Author (or script) at least **6 cases**, including:
  - One **right-answer-wrong-reason**: correct final state, but the trajectory skipped retrieval (passes final-state, fails trajectory).
  - One **unsafe-but-correct**: correct outcome reached via an unconfirmed destructive call (passes final-state, fails the safety predicate).
  - One **better-alternative-path**: a valid trajectory that differs from any reference sequence (must still pass — proves you didn't over-specify).

### 5. Report by quadrant

- Aggregate results into the four quadrants: (final-state pass/fail) x (trajectory pass/fail).
- Print a table; highlight the **"outcome correct, trajectory unsafe"** quadrant as the dangerous one.

## Starter guidance

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Step:
    kind: str              # "message" | "tool_call" | "tool_result"
    tool: str | None = None
    args: dict | None = None

Trajectory = list[Step]

@dataclass
class Case:
    case_id: str
    input: str
    final_state: dict
    required_tools: list[str]
    max_tool_calls: int

def score_final_state(run_state: dict, case: Case) -> bool:
    raise NotImplementedError  # compare to case.final_state

def score_trajectory(traj: Trajectory, case: Case) -> dict[str, bool]:
    raise NotImplementedError  # per-predicate results

def quadrant(final_ok: bool, traj_ok: bool) -> str:
    return {(True, True): "safe-correct",
            (True, False): "correct-but-unsafe",   # the dangerous one
            (False, True): "good-path-wrong-answer",
            (False, False): "broken"}[(final_ok, traj_ok)]
```

## Acceptance criteria

You can demonstrate that:

- The dataset is **versioned** and each case carries a final-state reference plus trajectory predicates (no golden transcript).
- The final-state and trajectory scorers run **independently** and report per-predicate trajectory results.
- Your "right-answer-wrong-reason" case **passes** final-state and **fails** trajectory; your "better-alternative-path" case **passes** trajectory despite differing from any reference sequence.
- The quadrant report clearly identifies at least one "outcome correct, trajectory unsafe" case.

## Reflection

In `NOTES.md`:

1. Which requirement in your domain needed *only* final-state eval, and which needed a trajectory predicate? Why?
2. Show a trajectory predicate you initially wrote too strictly (it failed a valid alternative path) and how you loosened it.
3. If you had to drop one lens for cost reasons, which would you keep for *your* domain, and what failure are you accepting?

## Stretch goals

- Curate three of your cases from **real traces** (from mod-305 observability output) rather than scripting them, and note what the real traces did that you wouldn't have invented.
- Add a **hidden split**: hold two cases out of all iteration and report the score gap between the iterated set and the hidden set as an overfitting check.
- Implement an order-*forbidding* predicate (e.g., "never `send_email` before `issue_refund`") and add a case that exercises it.
