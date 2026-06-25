# exercise-01: Token Economics Cost Model

**Estimated effort:** 3 hours

## Objective

Build a defensible token-economics cost model for a real fan-out agentic system, roll it up from per-task to per-tenant to run-rate, and use it to derive the system's **value threshold** — the per-task value the system must return to justify being an agent at all. Your deliverable is an architect's artifact: a cost-model worksheet plus a one-page memo a finance partner could read, not a tuned prompt.

## Background

This exercise covers material from:

- [Chapter 1 — Token Economics & Value Thresholds](../01-token-economics.md)

You are pricing the **architecture**, not running it. You may run the worksheet code to compute figures, but no live API calls are required. Use the illustrative price table from Chapter 1, and clearly label every number as illustrative-pending-verification.

## The system to price

Model this fan-out system (or substitute one of comparable shape from your own work and state the swap):

> **Competitive-intelligence agent.** A user submits a company name. An orchestrator decomposes the request into sub-questions (financials, product, hiring signals, recent news, regulatory). It fans out one worker per sub-question; each worker runs a short tool loop (search + read) and returns a distilled finding. The orchestrator synthesizes a brief. Expected volume: 800 briefs/day across 120 tenants, with a heavy-tenant tail.

## Prerequisites

- Comfort with a spreadsheet **or** the Python worksheet from Chapter 1.
- The workflow-vs-agent decision from [mod-301](../../mod-301-agentic-systems-foundations/README.md).

## Tasks

### 1. Per-call inventory

- Enumerate **every model call** one brief triggers: orchestrator decomposition, each worker (including its internal loop turns), and orchestrator synthesis.
- For each call, estimate input and output tokens. For worker loops, account for **re-sent history** — the transcript grows each turn (Chapter 1, fact 2).
- State your token estimates as assumptions with a one-line basis each ("worker base context ≈ 8k: system 1.5k + tools 2k + assignment 0.5k + first retrieved page 4k").

### 2. Per-task cost

- Compute the cost of one brief by summing all calls. Use the Chapter 1 worksheet or an equivalent spreadsheet.
- Produce the **cost shape**: what fraction of the per-task cost is decomposition vs. worker loops vs. synthesis? Identify the dominant term.

### 3. Roll up to per-tenant and run-rate

- Compute `daily_run_rate = task_cost × 800` and the annualized figure.
- Compute `per_tenant_cost` for a **mean** tenant and a **p95** tenant. Model the p95 tenant as triggering ~8× the mean tenant's briefs.
- Identify whether any tenant, at a plausible flat price point you choose, runs at **negative gross margin**.

### 4. Sensitivity analysis

- Re-run the model varying two parameters independently: **number of workers** (e.g., 5 → 8 → 12) and **worker loop turns** (e.g., 3 → 5 → 8).
- Produce a small table showing per-task cost at each setting. Note which parameter the cost is most sensitive to.

### 5. Derive the value threshold

- Estimate `value_per_task` — what a successful brief is worth (e.g., analyst-hours saved × loaded hourly rate). Defend the number in one or two sentences.
- Apply the threshold test from Chapter 1 with an explicit `safety_margin` (justify 3×–10×): does `value_per_task > task_cost × safety_margin` hold?
- State the verdict: **build the agent**, **build a cheaper variant**, or **do not build an agent** — and why.

## Starter guidance

Use this worksheet (the Chapter 1 model, ready to run) and fill in your estimates. Keep it in the repo next to your memo so it re-runs when prices change.

```python
"""exercise-01 token-economics worksheet.
Illustrative $/million-token prices — VERIFY against current provider pricing
before treating any output as a real budget."""
from dataclasses import dataclass

PRICES = {  # $ per 1,000,000 tokens — illustrative, replace with current
    "big":   {"in": 3.00, "out": 15.00},
    "small": {"in": 0.80, "out": 4.00},
}


@dataclass
class Call:
    model: str
    input_tokens: int
    output_tokens: int

    def cost(self) -> float:
        p = PRICES[self.model]
        return (self.input_tokens / 1e6) * p["in"] + (self.output_tokens / 1e6) * p["out"]


@dataclass
class Worker:
    model: str
    base_context: int
    growth_per_turn: int
    output_per_turn: int
    turns: int

    def cost(self) -> float:
        total, ctx = 0.0, self.base_context
        for _ in range(self.turns):
            total += Call(self.model, ctx, self.output_per_turn).cost()
            ctx += self.growth_per_turn + self.output_per_turn  # history re-sent
        return total


def task_cost(decompose: Call, workers: list[Worker], synth: Call) -> float:
    return decompose.cost() + sum(w.cost() for w in workers) + synth.cost()


def threshold_holds(value_per_task: float, cost: float, margin: float) -> bool:
    return value_per_task > cost * margin


if __name__ == "__main__":
    # ----- FILL IN YOUR ESTIMATES -----
    decompose = Call("big", 4_000, 500)
    workers = [Worker("big", 8_000, 1_500, 600, turns=4) for _ in range(5)]
    synth = Call("big", 9_000, 1_200)

    c = task_cost(decompose, workers, synth)
    print(f"per-task cost: ${c:.4f}")
    print(f"daily (800):   ${c * 800:,.2f}")
    print(f"annual:        ${c * 800 * 365:,.0f}")

    value_per_task, margin = 0.60, 5.0   # defend these in your memo
    print(f"threshold holds: {threshold_holds(value_per_task, c, margin)}")
```

A worked cost-model worksheet template (spreadsheet form) and the reference memo live in the [solutions repo](https://github.com/ai-engineering-curriculum/agentic-systems-architect-solutions).

## Deliverables

1. `cost-model.{py,xlsx,csv}` — the worksheet with your estimates, the cost shape, the sensitivity table, and the roll-ups.
2. `MEMO.md` — a one-page memo: per-task cost, daily/annual run-rate, the p95-tenant margin finding, the value-threshold verdict, and the assumptions list.

## Acceptance criteria

You can demonstrate that:

- Every model call a brief triggers is enumerated, with token estimates and a stated basis for each.
- Per-task cost is computed and **decomposed by phase** (decompose / workers / synthesis), naming the dominant term.
- Roll-ups exist for daily run-rate, annualized cost, and **mean vs. p95 tenant**, with a negative-margin check against a stated price point.
- A sensitivity table shows cost vs. worker count and loop turns, identifying the dominant driver.
- A value-threshold verdict is stated with an explicit `value_per_task`, a justified `safety_margin`, and a build / cheaper-variant / do-not-build conclusion.
- Every price and token figure is labeled illustrative-pending-verification.

## Reflection

In `NOTES.md`:

1. Which single parameter moved per-task cost the most? What does that imply about where the architecture should spend its optimization budget (preview of exercise-02)?
2. Did the p95 tenant change your verdict relative to the mean tenant? What pricing or capping change would you propose to protect margin?
3. Was there a sub-question (worker) whose value did **not** justify its cost? Would you cut it, downshift it, or batch it?

## Stretch goals

- Add a **retry/failure tax**: assume 12% of worker calls fail and are retried once. Re-derive per-task cost and re-check the threshold.
- Add a second tier of tenant and model **per-tenant margin** as a distribution, not two points; find the break-even brief volume.
- Pull **current** provider prices from the pricing page (see [resources.md](../resources.md)) and re-run; report how much your verdict moved.
