# Chapter 1 — Trajectory vs. Final-State Evaluation

Every evaluation of an agentic system answers one of two questions. **Final-state evaluation** asks: *was the outcome correct?* — the returned answer, the file written, the row inserted, the ticket resolved. **Trajectory evaluation** asks: *was the path correct?* — the sequence of messages, tool calls, and intermediate states the agent passed through to get there.

A single agent run produces both an outcome and a trajectory, so you can score it through either lens. The architect's job is to decide which lens each requirement needs — because each one catches failures the other is blind to.

```text
  user task
      │
      ▼
  ┌────────────────────────────────────────────────┐
  │  agent run                                      │
  │                                                 │
  │  msg → tool_call(search) → tool_result          │  ◀── trajectory
  │      → tool_call(fetch)  → tool_result          │      (the path)
  │      → msg → tool_call(write_file) → result     │
  │                                                 │
  └───────────────────────────┬────────────────────┘
                              ▼
                    final answer + world state        ◀── final state
                                                          (the outcome)
```

## What each lens catches — and misses

**Final-state eval** is the lens that matters to the user. They do not care how the agent got there; they care that the answer is right. It is also the cheaper lens to label: you compare an artifact to a reference, often with exact-match, a parser, or a single judge call. Its blind spot is the **right answer for the wrong reason**. An agent that guesses the capital of France without calling any tool "passes" final-state eval on that question — and then fails catastrophically on the next question where it needed to retrieve. Final-state eval cannot distinguish a robust strategy from a lucky one.

**Trajectory eval** is the lens that matters to *you*, the operator. It catches the failures that predict future incidents: the agent that retrieved nothing but answered anyway (hallucination risk), the agent that called a destructive tool it should have asked about first (safety risk), the agent that looped through eight redundant searches (cost and latency risk). Its blind spot is **over-specification**. If you assert the exact tool sequence, a *better* trajectory that solves the task differently fails your eval. You end up testing whether the agent imitates one reference path, not whether it solves the problem.

The discipline is to **score outcomes strictly and paths loosely**. Pin down what the final state must be. For the trajectory, assert *properties* ("retrieved before answering," "never called `delete_record` without a confirming step," "stayed under the tool-call budget"), not a verbatim transcript.

## Three flavors of trajectory match

When a requirement genuinely needs the path checked, pick the **loosest match that still catches the failure** you care about:

- **Exact match** — the trajectory must equal the reference step-for-step. Brittle; reserve it for short, deterministic flows where any deviation is a bug (e.g., a two-step "authenticate then call" handshake).
- **Order-allowing / superset match** — the required steps all appear, in a valid order, but extra steps are tolerated. Good default for "the agent must consult the policy doc before deciding," where you don't care what else it did.
- **Property / predicate match** — score a boolean function of the trajectory: `retrieved_before_answering(traj)`, `tool_budget_respected(traj, max=6)`, `no_unconfirmed_destructive_call(traj)`. This is the most robust and the one you'll reach for most. It decouples "did the agent behave correctly" from "did the agent reproduce a script."

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Step:
    kind: str          # "message" | "tool_call" | "tool_result"
    tool: str | None    # tool name when kind == "tool_call"

Trajectory = list[Step]

def tool_sequence(traj: Trajectory) -> list[str]:
    return [s.tool for s in traj if s.kind == "tool_call" and s.tool]

def required_steps_present(traj: Trajectory, required: list[str]) -> bool:
    """Order-allowing superset match: every required tool appears, in order,
    extra calls tolerated."""
    seq = tool_sequence(traj)
    i = 0
    for tool in seq:
        if i < len(required) and tool == required[i]:
            i += 1
    return i == len(required)

def retrieved_before_answering(traj: Trajectory) -> bool:
    """Property match: at least one retrieval tool ran before the final message."""
    saw_retrieval = False
    for s in traj:
        if s.kind == "tool_call" and s.tool in {"search", "fetch", "retrieve"}:
            saw_retrieval = True
        if s.kind == "message" and s is traj[-1]:
            return saw_retrieval
    return saw_retrieval
```

## Designing a dataset that serves both lenses

A dataset entry that supports both lenses carries, per case, an **input**, a **final-state reference**, and a set of **trajectory predicates** — not a golden transcript:

```text
case_id: kb-refund-007
input:   "Refund order 5512 and tell the customer."
final_state:
  refund_issued: true
  order_id: 5512
  customer_notified: true
trajectory_assertions:
  - required_steps_present: [lookup_order, issue_refund, send_email]
  - predicate: no_unconfirmed_destructive_call
  - predicate: tool_budget_respected(max=6)
labels:
  difficulty: medium
  capability: action-taking
```

Three rules keep such a dataset honest:

1. **Curate from real traces, not imagination.** Pull failing and near-miss runs from production traces (mod-305) and turn each into a case. Synthetic-only datasets test the failures you *thought* of, never the ones that actually bite.
2. **Hold a hidden split.** Keep some cases out of any iteration loop so you can detect when you've overfit the harness to a known set — the eval-set equivalent of a test set.
3. **Version it.** A score is only comparable against the same dataset version. Tag every eval run with the dataset commit; a "regression" against a silently changed dataset is noise, not signal.

## When to use which lens

| Requirement | Lens |
| --- | --- |
| "The answer must be factually correct." | Final-state |
| "The file must contain valid, parseable JSON." | Final-state |
| "The agent must retrieve evidence before answering." | Trajectory (predicate) |
| "It must never delete without a confirmation step." | Trajectory (predicate) |
| "It must finish within the tool-call budget." | Trajectory (predicate) |
| "It must follow the auth-then-call handshake exactly." | Trajectory (exact, short flow) |
| "The end answer must be right *and* grounded in retrieval." | Both, combined |

Most production requirements need **both**: a strict outcome check *and* one or two cheap trajectory predicates that guard against the right-answer-wrong-reason failure. Build the harness to return both scores per case so a single dashboard can show you "outcome correct but trajectory unsafe" — the most dangerous quadrant, because it passes naive eval and fails in production.

## Key takeaways

- **Final-state eval** scores the outcome (cheap, user-facing, blind to lucky-but-fragile paths); **trajectory eval** scores the path (operator-facing, catches the failures that predict incidents, blind to over-specification).
- Score **outcomes strictly, paths loosely** — assert trajectory *properties* (retrieved-before-answering, budget respected, no unconfirmed destructive call), not verbatim transcripts.
- Pick the **loosest trajectory match** that still catches the failure: exact → order-allowing superset → property/predicate, in increasing robustness.
- A good dataset case carries an input, a final-state reference, and trajectory predicates; curate it from **real traces**, hold a **hidden split**, and **version** it so scores are comparable.
- Most requirements need **both lenses** combined — the "outcome correct, trajectory unsafe" quadrant is the one that ships bugs.
