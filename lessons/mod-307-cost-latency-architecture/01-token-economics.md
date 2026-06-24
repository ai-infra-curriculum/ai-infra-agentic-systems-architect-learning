# Chapter 1 — Token Economics & Value Thresholds

Every agentic system has a unit cost, whether you have measured it or not. The architect's job is to measure it *at design time* — before the system ships — because in a multi-agent system the cost does not scale with the number of user requests. It scales with the number of **model calls**, and one user request can become dozens of model calls once an orchestrator fans out to workers, each worker loops over tools, and the orchestrator re-reads everything to synthesize. The gap between "one request" and "the tokens that request actually burns" is where unpriced architectures die.

This chapter builds the model that closes that gap, and then uses it to answer a design question that comes *before* any optimization: is this task even worth an agent?

## How tokens become dollars

Providers price two things separately, and the prices differ by roughly an order of magnitude:

- **Input tokens** — everything you send: system prompt, tools, conversation history, retrieved context, the user message.
- **Output tokens** — everything the model generates. Output is the expensive side, typically several times the per-token price of input.

Three structural facts make multi-agent cost non-obvious, and all three are architectural — you control them by how you draw the boxes:

1. **Input dominates token *count*; output dominates token *price*.** A worker prompt might be 8,000 input tokens and 600 output tokens. The input is 13× larger by count, but if output is priced 5× higher, the output still drives roughly a third of the line cost. You cannot reason about cost by looking at either column alone.
2. **History is re-sent every turn.** An agent loop with five tool-call turns re-sends the growing transcript on every turn. The same context tokens are paid for again and again. A 5-turn loop over a 10,000-token context does not cost 10,000 input tokens — it costs the running sum across turns, which is multiples of that. This is why caching (Chapter 2) is a lever and not a nicety.
3. **Fan-out multiplies everything.** An orchestrator that dispatches eight workers pays for eight worker contexts *plus* the orchestrator's own decomposition and synthesis calls. Anthropic has reported that their multi-agent research system used roughly **15× more tokens than a single chat interaction** — that multiplier is the thing you are pricing.

> **Pricing is provider- and time-specific.** Per-token prices, cache rates, and batch discounts change. This module teaches the *model*; always pull current numbers from the provider's pricing page (see [resources.md](resources.md)) before you commit a design to a budget. The worked numbers below are illustrative round figures, not quotes.

## The unit-economics model

Price one task end to end. Walk every model call the task triggers and sum its tokens. For a single call:

```text
call_cost = (input_tokens  / 1,000,000) * input_price_per_M
          + (output_tokens / 1,000,000) * output_price_per_M
```

For a fan-out task, the task cost is the sum over every call the task makes — orchestrator decomposition, each worker (including its internal loop turns), and orchestrator synthesis:

```text
task_cost = orchestrator_decompose
          + Σ  worker_i_cost      (each worker = its own loop of calls)
          + orchestrator_synthesize
```

Here is the model as runnable code. It is deliberately small — a worksheet you can keep in the repo next to the architecture diagram and re-run when prices or the fan-out shape change.

```python
"""Token-economics worksheet for a fan-out agentic task.

Prices are illustrative $/million-tokens. Replace with current provider
pricing before committing a design to a budget.
"""
from dataclasses import dataclass, field

# Illustrative price table ($ per 1,000,000 tokens). VERIFY against the
# provider pricing page before use — these are round teaching figures.
PRICES = {
    "big":   {"in": 3.00,  "out": 15.00},   # frontier / mid tier
    "small": {"in": 0.80,  "out": 4.00},    # cheap / fast tier
}


@dataclass
class Call:
    model: str          # key into PRICES
    input_tokens: int
    output_tokens: int

    def cost(self) -> float:
        p = PRICES[self.model]
        return (self.input_tokens / 1_000_000) * p["in"] + (
            self.output_tokens / 1_000_000
        ) * p["out"]


@dataclass
class Worker:
    """A worker is its own agent loop: `turns` calls over a growing context."""
    model: str
    base_context: int        # tools + system + assignment, re-sent each turn
    growth_per_turn: int     # tokens the transcript grows by per turn
    output_per_turn: int
    turns: int

    def calls(self) -> list[Call]:
        out = []
        ctx = self.base_context
        for _ in range(self.turns):
            out.append(Call(self.model, ctx, self.output_per_turn))
            ctx += self.growth_per_turn + self.output_per_turn  # history re-sent next turn
        return out

    def cost(self) -> float:
        return sum(c.cost() for c in self.calls())


@dataclass
class FanOutTask:
    decompose: Call
    workers: list[Worker]
    synthesize: Call
    label: str = "task"

    def cost(self) -> float:
        return (
            self.decompose.cost()
            + sum(w.cost() for w in self.workers)
            + self.synthesize.cost()
        )

    def report(self) -> str:
        worker_costs = [w.cost() for w in self.workers]
        return (
            f"[{self.label}] decompose=${self.decompose.cost():.4f}  "
            f"workers(n={len(self.workers)})=${sum(worker_costs):.4f}  "
            f"synthesize=${self.synthesize.cost():.4f}  "
            f"TOTAL=${self.cost():.4f}"
        )


if __name__ == "__main__":
    task = FanOutTask(
        label="research-question",
        decompose=Call("big", input_tokens=4_000, output_tokens=500),
        workers=[
            Worker("big", base_context=8_000, growth_per_turn=1_500,
                   output_per_turn=600, turns=4)
            for _ in range(6)
        ],
        synthesize=Call("big", input_tokens=9_000, output_tokens=1_200),
    )
    print(task.report())
    # Per-task cost * expected daily task volume = daily run-rate.
    daily_tasks = 2_000
    print(f"daily run-rate (n={daily_tasks}): ${task.cost() * daily_tasks:,.2f}")
    print(f"annualized: ${task.cost() * daily_tasks * 365:,.0f}")
```

The point of running this is not the exact dollar figure. It is the **shape** of the cost: where it concentrates (almost always the worker loops, because of re-sent history × fan-out), and how violently it moves when you change `turns` or the number of workers. An architect who has run this once stops treating "add another worker" as free.

## From per-task to per-tenant

Per-task cost is the atom; the molecules the business cares about are **per-tenant** and **per-run-rate**:

```text
daily_run_rate   = task_cost * tasks_per_day
per_tenant_cost  = task_cost * tasks_per_tenant_per_month
gross_margin      = price_charged - per_tenant_cost      (must stay positive)
```

The per-tenant view is where you catch the design that is fine in aggregate but lethal for your heaviest 5% of users. A flat-priced product with a fan-out agent is a structural invitation for a power user to run a negative-margin account. Architects surface this at design time by modeling a **p95 tenant**, not just the mean — the mean hides the tenant who triggers 40 tasks a day where everyone else triggers two.

## The value threshold: agent vs. workflow, priced

mod-301 taught you to choose a workflow over an agent on the basis of *control flow*. This module adds the second gate: even when the task shape *could* justify an agent, the **value it returns** must clear its cost. That is the value threshold.

```text
                         agentic value threshold
                                    │
   value per task   LOW             │            HIGH
   ──────────────────────────────────────────────────────▶
   cost per task
     LOW   │  workflow is fine;     │  agent is clearly
           │  agent is overkill     │  justified
     ──────┼────────────────────────┼──────────────────────
     HIGH  │  DO NOT BUILD —        │  agent justified IF
           │  cost exceeds value    │  value >> cost; else
           │  (the trap quadrant)   │  cap, cache, downshift
```

The test is a single inequality, applied per task:

```text
build the agent  ⇔  value_per_task  >  cost_per_task * safety_margin
```

- **`value_per_task`** is what a successful task is worth: revenue, hours of analyst time saved, a deflected support ticket, an averted incident. Put a number on it even if it is rough — a defended estimate beats an undefended vibe.
- **`safety_margin`** (e.g. 3×–10×) absorbs the things the happy-path cost model omits: retries, failed trajectories you still paid for, cost variance from looping, and the eval/observability overhead from mod-304 and mod-305.

When value comfortably exceeds cost, build the agent and spend the chapters that follow making it cheaper. When cost exceeds value, the correct architecture is often **not an agent at all** — it is a workflow, a cached single call, a cheaper model, or declining to automate the task. The most expensive line in any cost model is the one for a task that should never have been agentic.

## Worked judgment

*"Summarize each night's support tickets into a digest."* — High volume, low value per task, fully enumerable steps. `value_per_task` is small (a few minutes of a manager's reading). A fan-out agent here lands in the trap quadrant: real cost, thin value. The architecture is a **batched workflow** (Chapter 2), not an agent.

*"Triage a production incident: find the cause across logs, metrics, and recent deploys, and propose a fix."* — Open-ended path *and* high value (minutes of downtime cost more than the tokens). `value_per_task` dwarfs `cost_per_task` even at a 10× safety margin. This clears the threshold — build the agent, then cap and instrument it.

## Key takeaways

- Multi-agent cost scales with **model calls, not user requests**; fan-out and re-sent history can push token use to a large multiple (Anthropic reported ~15×) of a single chat.
- Price a task by **summing every call it triggers** — decomposition, each worker's loop, synthesis — then roll up to **per-tenant** and **run-rate**, modeling a **p95 tenant**, not just the mean.
- The **value threshold** is a hard design gate: build the agent only when `value_per_task > cost_per_task × safety_margin`; otherwise the correct architecture is a workflow, a cheaper call, or no automation.
- Always plug **current provider prices** into the model — the method is durable, the numbers are not.
