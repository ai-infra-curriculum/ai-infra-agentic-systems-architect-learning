# Chapter 3 — Cost Models & SLAs in the Reference Architecture

A cost model that lives in a spreadsheet on someone's laptop is a forecast. A cost model that lives *in the architecture* — as budgets the system enforces, SLAs the system is measured against, and kill switches the system trips — is a control plane. This chapter is about promoting cost and latency from numbers you estimated (Chapter 1) using levers you chose (Chapter 2) into **load-bearing components of the reference architecture itself**.

The shift is from *predicting* spend to *governing* it. An architecture that only predicts cost will, the first time real traffic deviates from the forecast, surprise you with a bill. An architecture that governs cost cannot — because the limits are wired in.

## Two budgets, not one

Every agentic task carries two budgets, and the reference architecture must make both explicit:

- **A cost budget** — a ceiling on tokens (and therefore dollars) per task, per tenant, and per run-rate. Derived from the value threshold of Chapter 1: a task worth $0.50 of saved analyst time gets a cost budget well under that, after the safety margin.
- **A latency budget** — a ceiling on wall-clock time per task, expressed as an SLA (e.g., "p95 < 8s for interactive triage; offline digests have no interactive SLA"). Derived from the user's tolerance, not from what the system happens to do.

These are inputs to the design, not outputs of it. You set the budgets from value and user tolerance, then design an architecture that *fits inside them* — caching, routing, batching, bounded fan-out, capped loops. When the cheapest viable design still busts the budget, that is a signal to revisit the value threshold (Chapter 1) or descope, not to quietly accept the overrun.

## A reference architecture with cost and latency wired in

Here is the shape. The point is the **control-plane components** — the meter, the budget guard, the kill switch — which a cost-blind architecture omits and a cost-aware one treats as first-class.

```text
                         ┌──────────────────────────────────────────┐
                         │              CONTROL PLANE               │
                         │  budget guard · token meter · kill switch│
                         └───────────────┬──────────────────────────┘
                                         │ enforces caps, reads meter
   request ─▶ ┌──────────┐   ┌───────────▼───────────┐   ┌──────────────┐
              │  router  │──▶│      orchestrator      │──▶│  synthesize  │─▶ result
              │ (big/sm) │   │ bounded fan-out (≤ N)  │   │ (big model)  │
              └──────────┘   └───────────┬───────────┘   └──────────────┘
                                  fan-out │  (each capped at T turns)
                          ┌───────────────┼───────────────┐
                          ▼               ▼               ▼
                    ┌───────────┐   ┌───────────┐   ┌───────────┐
                    │ worker    │   │ worker    │   │ worker    │
                    │ small +   │   │ small +   │   │ small +   │
                    │ cached    │   │ cached    │   │ cached    │  ← shared cached prefix
                    │ prefix    │   │ prefix    │   │ prefix    │
                    └───────────┘   └───────────┘   └───────────┘
                          │               │               │
                          └───────── meter every call ─────┘
                                         │
                         ┌───────────────▼──────────────┐
                         │  ASYNC / BATCH LANE (50% off) │  ← latency-tolerant work
                         │  nightly enrich · eval runs   │
                         └──────────────────────────────┘
```

Read the diagram as a set of cost-control decisions made structural:

- **Router at the edge** (Chapter 2, Lever 2) — every call enters on the cheapest viable model.
- **Bounded fan-out (≤ N)** — the orchestrator may emit at most N worker assignments; an unbounded decomposition is a runaway-cost incident waiting to happen.
- **Capped loops (≤ T turns)** — each worker's agent loop has a hard turn ceiling, because re-sent history (Chapter 1) makes an uncapped loop a cost time bomb.
- **Shared cached prefix** (Chapter 2, Lever 1) — fan-out siblings read one cached preamble.
- **The token meter** — every call reports `input_tokens`, `output_tokens`, `cache_read_input_tokens` to a meter the control plane reads in real time.
- **The budget guard and kill switch** — when a task (or tenant, or the system) crosses its cost budget, the guard degrades or halts it (next section).

## Guardrails: caps, degradation, and kill switches

A budget the system cannot enforce is a wish. Three enforcement mechanisms, in increasing severity:

1. **Hard caps (prevent).** Bound the things that multiply: max workers per task (N), max turns per loop (T), max output tokens per call. These are cheap to implement and stop the worst runaway before it starts. Every fan-out and every loop in the architecture must carry a cap; "the model will probably stop on its own" is not a budget.
2. **Graceful degradation (adapt).** When a task approaches its budget, downshift instead of failing: drop to a cheaper model, skip the optional refinement pass, return the best partial answer with a "truncated for budget" flag. Degradation keeps the system useful under pressure instead of brittle.
3. **Kill switches (stop).** When a tenant or the whole system blows a budget ceiling — a prompt-injection loop, a misconfigured client hammering the API, a pricing change that inverts margins overnight — a kill switch halts spend. Per-tenant kill switches contain the blast radius to one account; a global switch is the last line against a cost incident. Wire them to the same meter the budget guard reads.

Here is the budget guard as a small, testable component — the kind of thing that belongs in the reference architecture, not in a comment:

```python
"""Budget guard: enforces per-task and per-tenant cost ceilings against a
running token meter. Trips degradation, then a kill switch."""
from dataclasses import dataclass, field


@dataclass
class Budget:
    task_usd: float          # hard ceiling per task
    tenant_daily_usd: float  # hard ceiling per tenant per day
    degrade_at: float = 0.75  # fraction of task budget that triggers downshift


@dataclass
class Meter:
    spend_by_tenant: dict[str, float] = field(default_factory=dict)

    def record(self, tenant: str, call_cost: float) -> None:
        self.spend_by_tenant[tenant] = self.spend_by_tenant.get(tenant, 0.0) + call_cost


class BudgetExceeded(Exception):
    """Raised to trip the kill switch — caller halts the task/tenant."""


def guard(meter: Meter, budget: Budget, tenant: str,
          task_spend_so_far: float) -> str:
    """Return an action: 'proceed', 'degrade', or raise to kill."""
    if meter.spend_by_tenant.get(tenant, 0.0) >= budget.tenant_daily_usd:
        raise BudgetExceeded(f"tenant {tenant} hit daily ceiling")
    if task_spend_so_far >= budget.task_usd:
        raise BudgetExceeded(f"task for {tenant} hit per-task ceiling")
    if task_spend_so_far >= budget.task_usd * budget.degrade_at:
        return "degrade"   # downshift model / skip optional passes
    return "proceed"
```

The architectural claim is not that this exact code ships. It is that **a budget guard is a named component with a meter, ceilings, a degradation threshold, and a kill path** — and that the reference architecture shows where it sits and what it reads.

## SLAs: cost SLOs and latency SLOs together

You already write availability and latency SLOs. In an agentic system, **cost is an SLO too** — and the two interact, because the levers that cut cost can add latency (batching) and the moves that cut latency can add cost (parallelizing more workers, spending more tokens on a stronger model). State both, with targets:

| SLO | Target (example) | Enforced by |
| --- | --- | --- |
| Interactive latency | p95 < 8s, p99 < 15s | capped fan-out depth, sync lane, streaming |
| Offline latency | results within 24h | batch lane |
| Cost per task | < $0.04 mean, < $0.12 p95 | router + cache + per-task cap |
| Cost per tenant / day | < $5.00 | tenant meter + kill switch |
| Cache hit rate | > 70% of prefix tokens read from cache | prompt layout + TTL tuning |

Two disciplines make these real:

- **Measure the percentile, not the mean.** A mean cost-per-task of $0.04 can hide a p95 of $0.40 driven by tasks that fan out wide and loop deep. Budget and alert on **p95/p99**, because the tail is what generates the surprise bill and the SLA breach. The p95 tenant from Chapter 1 is the same idea applied to spend.
- **Wire SLOs to the meter and observability stack.** The cost SLO is only enforceable if the meter from this chapter feeds the dashboards and alerts from mod-305. An SLO with no telemetry behind it is a sentence in a doc.

## Key takeaways

- Promote cost and latency from **estimates** to **enforced budgets**: a cost budget and a latency SLA per task, per tenant, and per run-rate, set from value and user tolerance as design *inputs*.
- The cost-aware reference architecture has a **control plane** — token meter, budget guard, kill switch — plus structural caps: bounded fan-out (≤ N), capped loops (≤ T), shared cached prefix, and a split sync/batch lane.
- Enforce budgets with three mechanisms: **hard caps** (prevent), **graceful degradation** (adapt), and **kill switches** (stop), all reading the same meter.
- Treat **cost as an SLO** alongside latency; target and alert on **p95/p99**, not the mean, and wire every SLO to the observability stack (mod-305) so it is enforceable, not aspirational.
