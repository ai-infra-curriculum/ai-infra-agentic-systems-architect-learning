# Chapter 2 — Tool-Call Correctness: Selection and Argument Layers

The single richest source of agent failures is the tool call. An agent can read the task perfectly and still fail by reaching for the wrong tool, or by reaching for the right tool with the wrong arguments. Trajectory eval ([Chapter 1](01-trajectory-vs-final-state-eval.md)) tells you *a* tool call was wrong; this chapter is about pinning down *which layer* failed, because the fix differs completely.

Decompose every tool call into two independently checkable layers:

```text
   tool call = tool_name + arguments
                  │            │
                  ▼            ▼
            ┌──────────┐  ┌──────────────┐
            │ SELECTION│  │  ARGUMENTS   │
            │  layer   │  │    layer     │
            └──────────┘  └──────────────┘
   "right tool for      "called it correctly:
    the job?"            valid + faithful to intent?"
```

A selection failure means your tool descriptions or routing logic are wrong. An argument failure means your schema, parameter docs, or the model's extraction is wrong. Lumping them into one "tool call was bad" verdict throws away the diagnosis. Build the harness so every tool-call check reports which layer failed.

## The selection layer

Selection asks: **given the task and the available tools, did the agent pick the right one(s)?** Most of this layer is deterministic and cheap.

**Deterministic selection checks:**

- **Allowed/forbidden sets.** For a class of task, some tools are required, some are forbidden. A read-only question must never call a `write_*` or `delete_*` tool. Express this as set membership over the trajectory's tool calls — no model needed.
- **Reference tool set (order-allowing).** From a curated case, the set of tools that *should* appear. Use the order-allowing superset match from Chapter 1; do not over-constrain to an exact sequence.
- **Budget and redundancy.** Count calls per tool. Three identical `search` calls with the same query is a redundancy bug; flag it deterministically.

```python
from dataclasses import dataclass

@dataclass
class SelectionVerdict:
    layer: str = "selection"
    passed: bool = True
    reasons: list[str] | None = None

def check_selection(
    called_tools: list[str],
    *,
    required: set[str] | None = None,
    forbidden: set[str] | None = None,
    max_calls_per_tool: int | None = None,
) -> SelectionVerdict:
    reasons: list[str] = []
    called = set(called_tools)

    if forbidden and (hit := called & forbidden):
        reasons.append(f"called forbidden tool(s): {sorted(hit)}")
    if required and (missing := required - called):
        reasons.append(f"missing required tool(s): {sorted(missing)}")
    if max_calls_per_tool is not None:
        for tool in called:
            n = called_tools.count(tool)
            if n > max_calls_per_tool:
                reasons.append(f"{tool} called {n}x (max {max_calls_per_tool})")

    return SelectionVerdict(passed=not reasons, reasons=reasons or None)
```

**Where the LLM judge takes over:** deterministic sets can't tell you whether selection was *reasonable given an ambiguous task*. "Find out why the customer is upset" doesn't map to a fixed tool set; whether `read_tickets` or `read_call_logs` was the right first move is a judgment call. For these, give a judge the task, the available-tool descriptions, and the selected tool, and ask for a graded verdict (covered in [Chapter 3](03-llm-as-judge-rubrics.md)). Use the judge *only* where the deterministic check genuinely cannot decide — every judge call is cost, latency, and variance.

## The argument layer

Selection can be perfect and the call still wrong because the arguments are. This layer has three checks, in increasing cost:

**1. Schema validity (deterministic, always run).** Does the argument object satisfy the tool's parameter schema — required fields present, types correct, enums in range, formats valid? This is `jsonschema` / Pydantic validation. It catches the most common and most embarrassing failures (missing required field, string where an int was expected) for near-zero cost. Run it on every tool call in every eval.

```python
from jsonschema import Draft202012Validator

@dataclass
class ArgumentVerdict:
    layer: str = "arguments"
    passed: bool = True
    reasons: list[str] | None = None

def check_arguments_schema(args: dict, schema: dict) -> ArgumentVerdict:
    errors = sorted(Draft202012Validator(schema).iter_errors(args),
                    key=lambda e: e.path)
    reasons = [f"{'/'.join(map(str, e.path)) or '<root>'}: {e.message}"
               for e in errors]
    return ArgumentVerdict(passed=not reasons, reasons=reasons or None)
```

**2. Value constraints (deterministic where you can express them).** Schema-valid is not the same as correct. A `limit` of `1000000` is schema-valid and operationally insane. A date range with `start > end` parses fine and means nothing. An `order_id` that doesn't match the order referenced in the task is valid JSON and wrong. Encode the constraints you can as predicates: cross-field invariants, value bounds, and — powerfully — **consistency with the task input** (the `order_id` in the call equals the `order_id` in the prompt). These catch faithfulness failures without a model.

**3. Argument faithfulness (LLM judge, where extraction is fuzzy).** The hard case: did the agent extract the right values from a natural-language task? "Email the two managers from the Phoenix office" requires resolving "the two managers" and "Phoenix office" into concrete recipient IDs. No schema or deterministic predicate can confirm those IDs are the *right* people. Here you hand a judge the task, the tool, and the arguments, and ask: are these arguments a faithful, complete reading of what the task asked for? This is the one argument check that needs a judge — and only because the ground truth lives in free text.

## Layering the checks: deterministic first, judge last

The architecture is a **cascade**: cheap deterministic checks gate the expensive judge. Never spend a judge call on a call that already failed schema validation — the diagnosis is already complete.

```python
def evaluate_tool_call(call, schema, *, value_checks, judge=None, task=None):
    """Returns per-layer verdicts; short-circuits before the judge."""
    verdicts = {}

    # Selection (deterministic) handled at trajectory level; assume passed here.
    schema_v = check_arguments_schema(call.args, schema)
    verdicts["arguments_schema"] = schema_v
    if not schema_v.passed:
        return verdicts  # malformed call — no point judging faithfulness

    value_v = run_value_checks(call.args, value_checks, task=task)
    verdicts["arguments_values"] = value_v
    if not value_v.passed:
        return verdicts

    # Only fuzzy extraction reaches the judge.
    if judge is not None and call.needs_faithfulness_judge:
        verdicts["arguments_faithfulness"] = judge.score_arguments(
            task=task, tool=call.tool, args=call.args,
        )
    return verdicts
```

This cascade is also your cost-control mechanism. On a 500-case suite where every case has 3–8 tool calls, judging every argument would be thousands of judge calls per eval run. Deterministic checks dispose of the vast majority — schema and value checks catch most argument bugs — and the judge handles only the genuinely fuzzy minority. The result is a harness that is both **diagnostic** (you know the exact layer that failed) and **affordable** (you don't pay a model to confirm what `jsonschema` already proved).

## Reading the verdicts

Because every check is tagged by layer, you can aggregate per-layer pass rates across the suite. That aggregation is the diagnosis:

| Symptom | Likely cause | Where to fix |
| --- | --- | --- |
| Selection pass rate dropped | Tool descriptions overlap / are ambiguous | Tool descriptions, routing prompt |
| Schema failures spiking | Schema changed, or model misreads it | Parameter schema, examples in tool doc |
| Value-constraint failures | Model ignores bounds / cross-field rules | Parameter docs, add constraints to prompt |
| Faithfulness failures | Extraction from NL is unreliable | Task framing, few-shot, stronger model |

A blended "tool accuracy: 82%" number tells you nothing actionable. The layered breakdown tells you exactly which prompt or schema to edit.

## Key takeaways

- Split every tool call into a **selection** layer (right tool?) and an **arguments** layer (called correctly?), and report which layer failed — the fix differs entirely.
- Selection is mostly **deterministic**: required/forbidden sets, order-allowing reference sets, budget and redundancy counts. Use a judge only for genuinely ambiguous task→tool mapping.
- Arguments check in three rising tiers: **schema validity** (always, near-free) → **value constraints / task-consistency** (deterministic where expressible) → **faithfulness** (LLM judge, only when ground truth lives in free text).
- Layer the checks as a **cascade**: cheap deterministic gates run first and short-circuit, so the judge only sees calls that need it — that's both the diagnosis and the cost control.
- Aggregate **per-layer** pass rates; a single blended "tool accuracy" hides the one number that would tell you what to fix.
