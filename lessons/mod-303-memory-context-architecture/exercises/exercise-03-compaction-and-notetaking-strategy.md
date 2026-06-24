# exercise-03: Compaction and Note-Taking Strategy

**Estimated effort:** 3 hours

## Objective

Design and **build** the two strategies that keep a long-horizon agent's window clean: a **compaction** mechanism (summarize history on a trigger) and a **structured note-taking** scratchpad (durable working state outside the window). Then *prove* they preserve task-critical state — that compacting hard and offloading to notes doesn't silently drop the user's goal, an in-flight action, or an unresolved error. This is the mitigation half of [exercise-01](exercise-01-context-budget-and-rot-analysis.md)'s diagnosis.

## Background

This exercise covers material from:

- [Chapter 1 — Context Engineering as a Token Budget](../01-context-engineering-strategies.md)
- [Chapter 2 — Diagnosing and Mitigating Context Rot](../02-context-rot.md)

Use any model provider and a multi-turn agent (yours, or the loop from exercise-01).

## Prerequisites

- An agent loop with a growing conversation history.
- Token counting (to drive a threshold trigger).
- A way to persist a small file or object between turns (for the notes store).

## Tasks

### 1. Define the fidelity contract

Before writing any compaction, write down what compaction must **never** lose. At minimum: the user's actual goal, IDs / file paths / handles in play, irreversible actions already taken, and unresolved errors. This contract is the spec your compaction is tested against. State it explicitly in `NOTES.md`.

### 2. Build the compaction trigger

Implement compaction that fires on a **token threshold** (when conversation history exceeds its budgeted slice from [Chapter 1](../01-context-engineering-strategies.md)). On fire:

- Summarize the spanned history into a compact form that satisfies the fidelity contract.
- Replace the raw span in the window with the summary.
- Persist the raw span to a store (so compaction is lossy *in context*, recoverable *from store*).

Make the threshold and the summarization prompt explicit and tunable.

### 3. Build the structured-notes store

Implement a **typed** note store — not free text. A task ledger works well:

```text
{ step_id, description, status: todo|doing|done|blocked, artifact, blocker }
```

The agent writes/updates the ledger as it works and reads back only the slice it needs (e.g., open and blocked items). The window holds a pointer and a short summary; the ledger holds the detail.

### 4. Prove fidelity preservation

This is the graded core. Construct a run where the fidelity contract is at risk — e.g., the user states a goal at turn 2, an irreversible action happens at turn 6, an error is opened at turn 9, and compaction fires at turn 12. After compaction:

- Assert the goal, the irreversible action, and the open error are all still recoverable from window + notes.
- Show what the agent's window looks like before vs. after compaction (token counts and the preserved facts).

Write this as an automated check, not a manual read.

### 5. Demonstrate the reset

Show **re-anchoring** ([Chapter 2](../02-context-rot.md)): hard-compact, then reconstruct a clean working context from the structured notes (the goal + open ledger items) as if starting fresh. Confirm the agent can continue the task from the reconstructed anchor.

## Starter guidance

```python
# compaction + typed notes — skeleton
from dataclasses import dataclass, field
from typing import Literal

Status = Literal["todo", "doing", "done", "blocked"]

@dataclass
class LedgerItem:
    step_id: int
    description: str
    status: Status = "todo"
    artifact: str = ""
    blocker: str = ""

@dataclass
class NotesStore:
    goal: str = ""
    items: list[LedgerItem] = field(default_factory=list)
    def open_items(self) -> list[LedgerItem]:
        return [i for i in self.items if i.status in ("todo", "doing", "blocked")]

HISTORY_BUDGET_TOKENS = 8000  # tune to your Chapter 1 budget

def should_compact(history_tokens: int) -> bool:
    return history_tokens > HISTORY_BUDGET_TOKENS

def compact(history: list[dict], notes: NotesStore) -> tuple[str, list[dict]]:
    """Return (summary, raw_span_to_persist). Summary MUST satisfy the
    fidelity contract: goal, IDs/paths, irreversible actions, open errors."""
    raise NotImplementedError

# Fidelity test (the graded part): seed a transcript with a goal, an
# irreversible action, and an open error; compact; then assert each is
# still recoverable from (summary + notes.open_items()).
```

```text
FIDELITY CONTRACT (fill before coding)
  MUST preserve through any compaction:
    □ user goal: ________________________________
    □ IDs / paths / handles in play
    □ irreversible actions already taken
    □ unresolved errors / blockers
  MAY drop: verbose tool scaffolding, resolved sub-steps, abandoned reasoning
```

## Acceptance criteria

You can demonstrate that:

- A written **fidelity contract** exists and your compaction is tested against it.
- Compaction **fires on a token threshold**, replaces raw history with a summary, and **persists the raw span** to a store.
- A **typed notes store** (not free text) holds working state, and the agent reads back only the needed slice.
- An **automated fidelity check** proves the goal, an irreversible action, and an open error survive a compaction event.
- You can **re-anchor**: reconstruct a clean working context from the notes and continue the task.

## Reflection

In `NOTES.md`:

1. What did your first compaction prompt drop that the fidelity contract required? How did you fix it — prompt, schema, or trigger?
2. Token counts before/after compaction: how much window did you reclaim, and at what summarization cost?
3. Free-text notes vs. your typed ledger — give a concrete case where the type structure prevented a failure free text would have allowed.

## Stretch goals

- Add a **boundary-based** trigger (compact at task-phase transitions) alongside the threshold trigger and compare which preserves fidelity better on your task.
- Make compaction **reversible on demand**: give the agent a tool to fetch the raw pre-compaction span from the store when it needs detail it summarized away.
- Run the exercise-01 quality-vs-length curve **with this compaction on** and show the rot peak moving right — closing the loop between diagnosis and mitigation.
