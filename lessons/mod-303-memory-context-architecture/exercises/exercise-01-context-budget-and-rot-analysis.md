# exercise-01: Context Budget and Rot Analysis

**Estimated effort:** 3 hours

## Objective

Take a long-horizon agent run, *measure* how it spends the context window, build a defensible **token budget** for it, and find the point where **context rot** sets in. You'll leave with two architect artifacts: a filled budget worksheet and a quality-vs-window-length curve that locates the rot — the evidence base for every mitigation decision in this module.

This is a measurement and design exercise, not a build-from-scratch one. You instrument an existing agent (yours from a prior module, or a small reference loop) rather than writing a new agent.

## Background

This exercise covers material from:

- [Chapter 1 — Context Engineering as a Token Budget](../01-context-engineering-strategies.md)
- [Chapter 2 — Diagnosing and Mitigating Context Rot](../02-context-rot.md)

Use any model provider. You need a multi-turn agent (10+ turns) doing a task with growing context — a research agent, a coding agent, or a multi-step tool-using agent all work. If you don't have one, a loop that repeatedly retrieves documents and answers questions is enough.

## Prerequisites

- An agent loop you can instrument (log per-turn token counts).
- A token counter for your model (the provider's tokenizer or token-count endpoint).
- A quality signal: task success, or an LLM-judge score (a simple 1–3 rubric is fine).
- A small spend cap — the long runs cost tokens.

## Tasks

### 1. Instrument the window

Log, **per turn**, at minimum:

- `turn_index`
- `tokens_in_window` (total prompt tokens sent that turn)
- `tokens_by_component` — break the window into the Chapter 1 line items: system+tools (stable), working set, retrieved-this-turn, conversation history, sub-agent returns.
- `stale_fraction` — the fraction of the window that is history older than 5 turns.

Persist these as a per-turn record (CSV or JSON lines) so you can plot them.

### 2. Build the budget worksheet

Using the [Chapter 1 worksheet](../01-context-engineering-strategies.md#starter-guidance--the-context-budget-worksheet), fill the budget for *your* agent against *your* model's window. Set a steady-state target (start at ~65% of capacity). For every line item, write its strategy, a cache flag, and an **eviction rule**. Compare your designed budget to what you actually measured in Task 1 — note where reality blew the budget.

### 3. Find the rot

Run the agent on tasks of **increasing length** (e.g., 5, 10, 20, 40 turns, or increasing retrieved-context size). Plot your quality signal against `tokens_in_window` (or turn index). Identify:

- Whether quality is flat, rising, or **peaks and falls** (rot).
- If it falls, the approximate window occupancy where the peak is — your empirical occupancy ceiling.

### 4. Probe with distractors

Run a needle-in-a-haystack test *with distractors*: place one true fact and two plausible-but-wrong facts at different depths in a long window, then ask for the true fact. Vary the depth (start, middle, end). Record where the agent retrieves the true fact vs. a distractor vs. nothing. Confirm (or refute) the lost-in-the-middle effect for your model.

### 5. Write the rot budget

From the evidence in Tasks 3–4, fill the [Chapter 2 rot budget](../02-context-rot.md#the-architects-rot-budget): an occupancy ceiling, a staleness ceiling, a critical-fact placement rule, and the triggers that fire when each is crossed. Each number must trace back to something you measured.

## Starter guidance

```python
# instrument.py — minimal per-turn window telemetry
import json
from dataclasses import dataclass, asdict

@dataclass
class TurnRecord:
    turn_index: int
    tokens_in_window: int
    tok_system_tools: int
    tok_working_set: int
    tok_retrieved: int
    tok_history: int
    tok_subagent: int
    stale_fraction: float        # history older than 5 turns / tokens_in_window
    quality: float | None        # filled post-hoc by judge or success check

def log_turn(rec: TurnRecord, path="run.jsonl") -> None:
    with open(path, "a") as f:
        f.write(json.dumps(asdict(rec)) + "\n")

# count_tokens(text) -> int  : use your provider's tokenizer / count endpoint
# After the run, load run.jsonl and plot quality vs tokens_in_window.
```

```text
Distractor probe layout (one long window):
  [ ...filler... | NEEDLE: "the launch code is 4471" | ...filler...
    | DISTRACTOR: "the backup code is 9920"
    | ...filler... | DISTRACTOR: "an old launch code was 1130" | ...filler... ]
  Ask: "What is the launch code?"  → expect 4471, regardless of depth.
  Sweep NEEDLE depth across 10% / 50% / 90% of the window.
```

You do **not** need to *fix* the rot here — that's exercise-03. This exercise is diagnosis: measure, budget, locate.

## Acceptance criteria

You can demonstrate that:

- You have **per-turn telemetry** with the window broken into Chapter 1 components and a stale-fraction.
- You produced a **filled budget worksheet** for your agent with an eviction rule per line item, and you can point to where measured usage diverged from the budget.
- You have a **quality-vs-window-length plot** and can state whether rot occurs and at roughly what occupancy.
- Your **distractor probe** shows how depth affects retrieval (lost-in-the-middle confirmed or refuted with data).
- You produced a **rot budget** whose every threshold traces to a measurement.

## Reflection

In `NOTES.md`:

1. Where did your *measured* window spend differ most from your *designed* budget? What does that tell you about which line item to attack first?
2. At what occupancy did quality peak? How much headroom does that leave below your model's hard limit, and why is "fill the window" the wrong default?
3. Which rot source (dilution, distraction, self-conditioning) does your distractor probe most implicate for your agent?

## Stretch goals

- Repeat the quality-vs-length curve with **compaction on** (summarize history past a threshold) and overlay the two curves to show compaction moving the peak right.
- Add a second model with a different window size and compare the rot onset — does a bigger window just move the cliff, or remove it?
- Turn the distractor probe into a **CI check**: assert the agent retrieves the true fact at all three depths, and fail the build if it doesn't (a bridge to [mod-304](../../mod-304-evaluation-harnesses/README.md)).
