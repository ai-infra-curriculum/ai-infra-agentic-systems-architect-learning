# exercise-03: Cost–Latency–Quality Budgets

**Estimated effort:** 4 hours

## Objective

Close the module by turning your cost model (exercise-01) and levers (exercise-02) into a **governed reference architecture**: explicit per-use-case budgets that resolve the cost–latency–quality trade-off, an SLA table, the control-plane components (meter, budget guard, kill switch) that enforce the budgets, and the architecture decision records (ADRs) that defend the trades. This is the capstone artifact — the thing you would hand to an implementing team and defend to both a CFO and an SRE.

## Background

This exercise covers material from:

- [Chapter 3 — Cost Models & SLAs in the Reference Architecture](../03-cost-models-and-slas.md)
- [Chapter 4 — The Cost–Latency–Quality Triangle](../04-cost-latency-quality-budgets.md)
- exercise-01 (cost model) and exercise-02 (levers) — you build directly on both.

## The system to govern

Use the competitive-intelligence agent from exercise-01, now with three distinct surfaces (or substitute your own system with at least three surfaces of differing latency/quality needs):

> - **Interactive brief** — a user submits a company and waits for a brief. High volume, tight latency.
> - **Watchlist refresh** — nightly re-run of briefs for every watched company. Bulk, latency-tolerant.
> - **Deep-dive analysis** — an analyst requests an exhaustive, high-stakes report. Low volume, quality-maximal.

## Prerequisites

- exercise-01 (per-task and per-tenant cost model).
- exercise-02 (caching/routing/batching levers and the routed diagram).
- Familiarity with writing ADRs and SLOs.

## Tasks

### 1. Set the budgets per surface

- For each of the three surfaces, write a **budget** that fixes two corners of the triangle and names the third as the free variable (Chapter 4): e.g., interactive = "cost and latency fixed, quality floats."
- Express the fixed corners as numbers: a cost ceiling ($/task), a latency target (p95 seconds, or "24h, no interactive SLA"), and/or a quality floor (eval score).
- Justify each budget from **value and user tolerance**, not from what the system happens to do.

### 2. Resolve the trade and pick the levers

- For each surface, state the design point and which exercise-02 levers apply to hit the budget (e.g., interactive → small model + cache + capped fan-out; watchlist → batch lane + small model + cache; deep-dive → big model, deep loops).
- Where a budget cannot be met by the cheapest viable design, apply Chapter 4's honest moves: relax a corner's neighbor, revisit the value threshold, or descope — and record which.

### 3. Build the SLA table

- Produce a combined **cost + latency SLA table** (Chapter 3) covering all three surfaces, with targets on **p95/p99**, not the mean.
- Include a cache-hit-rate SLO and a per-tenant daily cost SLO, and name what **enforces** each row (cap, router, meter, batch lane).

### 4. Wire the control plane

- Draw (or extend the exercise-02 diagram into) a **governed reference architecture** showing the control plane: token meter, budget guard, kill switch, plus the structural caps (bounded fan-out ≤ N, capped loops ≤ T).
- Specify the **three enforcement mechanisms** for this system: the hard caps (give N and T), the graceful-degradation behavior (what downshifts/drops at the degrade threshold), and the kill-switch triggers (per-tenant and global).
- Adapt the Chapter 3 budget-guard component to your numbers (fill in `task_usd`, `tenant_daily_usd`, `degrade_at`).

### 5. Write the ADRs

- Write **three ADRs**, one per material trade-off, each in the format below. At minimum: (a) the model-routing decision for the interactive surface, (b) the batch-lane decision for the watchlist, (c) one budget you could *not* meet and how you resolved it.

## Starter guidance

ADR template — keep each to a page:

```text
# ADR-00N: <decision title>

## Status
Accepted | Proposed | Superseded

## Context
What forced the decision? Which budget/SLA/triangle corner is in tension?
Reference the exercise-01 cost figure and exercise-02 lever this concerns.

## Decision
The position chosen in the cost–latency–quality triangle, stated as the
two fixed corners and the floating one, with numbers.

## Levers & preconditions
Which of caching / routing / batching, and the precondition each relies on.

## Evidence
- Cost-model output proving it fits the cost box (exercise-01).
- Eval result proving the cheaper/faster choice clears the quality bar (mod-304).

## Accepted trade-off
What we gave up, stated plainly. (e.g., "p95 quality dips on multi-hop
questions in exchange for halving cost; deep-dive routes to the big-model lane.")

## Consequences
Caps, SLAs, and guardrails this implies; what the implementing team must build.
```

Budget guard to adapt (from Chapter 3) — fill in your ceilings:

```python
from dataclasses import dataclass, field


@dataclass
class Budget:
    task_usd: float           # per-task ceiling  (FILL IN per surface)
    tenant_daily_usd: float   # per-tenant/day ceiling (FILL IN)
    degrade_at: float = 0.75  # fraction of task budget that triggers downshift


@dataclass
class Meter:
    spend_by_tenant: dict[str, float] = field(default_factory=dict)

    def record(self, tenant: str, call_cost: float) -> None:
        self.spend_by_tenant[tenant] = self.spend_by_tenant.get(tenant, 0.0) + call_cost


class BudgetExceeded(Exception):
    """Trips the kill switch."""


def guard(meter: Meter, b: Budget, tenant: str, task_spend: float) -> str:
    if meter.spend_by_tenant.get(tenant, 0.0) >= b.tenant_daily_usd:
        raise BudgetExceeded(f"{tenant} hit daily ceiling")
    if task_spend >= b.task_usd:
        raise BudgetExceeded(f"task for {tenant} hit per-task ceiling")
    if task_spend >= b.task_usd * b.degrade_at:
        return "degrade"
    return "proceed"
```

Worked budgets, the full SLA table, the governed-architecture diagram, and reference ADRs live in the [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

## Deliverables

1. `budgets.md` — per-surface budgets (two fixed corners + floating), with value/tolerance justification.
2. `sla.md` — the combined cost + latency SLA table on p95/p99, with the enforcer named per row.
3. `governed-architecture.{txt,png}` — the reference architecture with the control plane and structural caps.
4. `budget-guard.py` — the guard adapted to your ceilings.
5. `adr/ADR-001…003.md` — three ADRs in the template format.

## Acceptance criteria

You can demonstrate that:

- Each of the three surfaces has a **budget that fixes two triangle corners and names the floating one**, with numeric ceilings justified by value and user tolerance.
- An **SLA table** targets cost and latency on **p95/p99** (not the mean), includes a cache-hit and a per-tenant cost SLO, and names the enforcer of each row.
- The reference architecture shows a **control plane** (meter, budget guard, kill switch) and structural caps with concrete N and T values.
- The three enforcement mechanisms are specified: hard caps, a graceful-degradation behavior, and per-tenant + global kill-switch triggers.
- Three **ADRs** are written in the template format, each naming the accepted trade-off plainly and citing cost-model and eval evidence.
- At least one ADR documents a budget that could **not** be met and the honest resolution chosen.

## Reflection

In `NOTES.md`:

1. Which surface was hardest to fit in its budget, and which corner did you end up floating? Would the budget owner agree with the corner you chose?
2. Where do cost and latency SLOs **conflict** in this system (a lever that cuts one and worsens the other)? How did the budget resolve it?
3. If you had to defend the interactive surface's model-routing decision in an incident review after a quality complaint, does your ADR's evidence hold up?

## Stretch goals

- Add a **runtime budget-vs-actual** monitor: extend the meter to alert when p95 cost-per-task drifts above the SLO, and define the runbook response.
- Model a **price-shock scenario**: the provider raises output-token price 2×. Which budgets break, which kill switches trip, and what is your degradation plan?
- Add a fourth surface (e.g., a free-tier preview) and show how its budget forces a *different* triangle position than the paid interactive brief.
