# exercise-02: Build a Tool-Call Correctness Harness

**Estimated effort:** 3 hours

## Objective

Build a tool-call correctness harness that checks every tool call at **two layers** — selection (right tool?) and arguments (called correctly?) — using a **cascade** of deterministic checks gating an LLM judge. By the end you'll have a harness that reports *which layer* failed for every call, and a per-layer pass-rate breakdown that points directly at the prompt or schema to fix.

## Background

This exercise covers material from:

- [Chapter 2 — Tool-Call Correctness: Selection and Argument Layers](../02-tool-call-correctness.md)

Reuse the agent and trace format from [exercise-01](exercise-01-trajectory-eval-design.md). You need tool calls with names and structured arguments, plus a JSON Schema for each tool's parameters.

## Prerequisites

- A set of tools each with a JSON Schema for its parameters (3+ tools, at least one with a constrained field — enum, bounded int, or a cross-field rule).
- Traces containing tool calls (`tool` + `args`).
- A JSON Schema validator (e.g., `jsonschema`) and an LLM client for the judge.
- A small spend cap for judge calls.

## Tasks

### 1. Selection-layer checks (deterministic)

- Implement `check_selection(called_tools, required, forbidden, max_calls_per_tool)`:
  - **Forbidden set** — flag any call to a tool that must not appear for the task class (e.g., a read-only question calling a `write_*`/`delete_*` tool).
  - **Required set** — flag missing required tools (order-allowing).
  - **Redundancy/budget** — flag a tool called more than `max_calls_per_tool` times.
- Return a `SelectionVerdict` tagged `layer="selection"` with human-readable reasons.

### 2. Argument-layer checks (deterministic, two tiers)

- **Schema validity:** validate each call's `args` against its tool schema; return every error path. Run this on *every* call.
- **Value constraints:** implement at least three predicates — a bound (`limit <= MAX`), a cross-field invariant (`start <= end`), and a **task-consistency** check (an ID in the args matches the ID referenced in the task input).
- Return an `ArgumentVerdict` tagged `layer="arguments"` per tier.

### 3. Argument faithfulness (LLM judge)

- For calls where the right value must be *extracted from natural language* (e.g., "email the two managers from Phoenix"), call a judge with the task, the tool, and the args, and ask for a graded faithfulness verdict.
- Mark which calls need the judge (`needs_faithfulness_judge`); do **not** judge calls that can be checked deterministically.

### 4. Wire the cascade

- Implement `evaluate_tool_call` so the checks run cheapest-first and **short-circuit**: a schema failure stops before value checks; a value failure stops before the judge. A judge call must never run on a call that already failed a deterministic check.

### 5. Per-layer aggregation

- Run the harness over your trace set and report **per-layer pass rates** (selection, arguments-schema, arguments-values, arguments-faithfulness) — not one blended "tool accuracy" number.
- Include a count of judge calls *saved* by the cascade (calls disposed of deterministically).

## Starter guidance

```python
from dataclasses import dataclass
from jsonschema import Draft202012Validator

@dataclass
class Verdict:
    layer: str
    passed: bool
    reasons: list[str] | None = None

def check_arguments_schema(args: dict, schema: dict) -> Verdict:
    errors = sorted(Draft202012Validator(schema).iter_errors(args),
                    key=lambda e: e.path)
    reasons = [f"{'/'.join(map(str, e.path)) or '<root>'}: {e.message}"
               for e in errors]
    return Verdict("arguments_schema", not reasons, reasons or None)

def evaluate_tool_call(call, schema, value_checks, judge=None, task=None):
    verdicts = {}
    sv = check_arguments_schema(call.args, schema)
    verdicts["arguments_schema"] = sv
    if not sv.passed:
        return verdicts                      # short-circuit: malformed
    vv = run_value_checks(call.args, value_checks, task=task)
    verdicts["arguments_values"] = vv
    if not vv.passed:
        return verdicts                      # short-circuit: invalid values
    if judge and call.needs_faithfulness_judge:
        verdicts["arguments_faithfulness"] = judge.score_arguments(
            task=task, tool=call.tool, args=call.args)
    return verdicts
```

## Acceptance criteria

You can demonstrate that:

- Every verdict is **tagged by layer**; a call that picked the wrong tool and a call that malformed its args produce *different* layer verdicts.
- Schema validity runs on **every** call; the judge runs **only** on calls flagged for faithfulness.
- The cascade **short-circuits**: you can show a count of judge calls saved because a deterministic check failed first.
- The final report is a **per-layer pass-rate breakdown**, and you can name which prompt/schema each failing layer points you to fix.

## Reflection

In `NOTES.md`:

1. Give a tool call that was schema-valid but value-wrong, and the deterministic predicate that caught it. Why couldn't schema validation catch it?
2. Which of your value checks could you express deterministically that you initially assumed needed a judge?
3. How many judge calls per eval run did the cascade save versus judging every argument? Estimate the cost difference at suite scale.

## Stretch goals

- Add a selection-layer **judge** for one genuinely ambiguous task (no fixed correct tool set) and calibrate it lightly against your own labels on 5 cases.
- Inject a schema change (rename a required field) and show the harness localizes the spike to `arguments_schema`, not selection.
- Emit the per-call verdicts in a format your [exercise-04](exercise-04-eval-gated-release-pipeline.md) gate can consume directly.
