# Chapter 4 — The Cost–Latency–Quality Triangle

Cost, latency, and quality form a triangle, and you cannot maximize all three at once. Spend more tokens on a stronger model and quality rises but so does cost; parallelize harder and latency falls but cost rises; batch aggressively and cost falls but latency rises; cap loops to save money and quality on the hard tail can drop. Every architecture sits somewhere inside this triangle. The amateur move is to pretend you can have all three; the architect's move is to **choose your position explicitly, write down the budget, and defend the trade**.

This chapter is where the three previous ones converge. Chapter 1 priced the system, Chapter 2 gave you the levers, Chapter 3 wired in the budgets and SLAs — here you resolve the conflicts *between* them, under explicit budgets, per use case.

## The triangle

```text
                         QUALITY
                       (accuracy,
                       completeness,
                     reasoning depth)
                          ▲
                         ╱ ╲
                        ╱   ╲
                       ╱     ╲
            pick a    ╱  your  ╲   every design
            point ──▶╱  design  ╲◀── is a point
            inside  ╱   sits     ╲   inside, never
                   ╱    HERE      ╲  a corner
                  ╱_______________╲
              COST                 LATENCY
          ($ / task)            (wall-clock)
```

The triangle is not a law of physics with a fixed exchange rate — the levers from Chapter 2 *bend* it (caching buys cost back without touching quality; routing buys cost back on calls that never needed the big model). But the levers have limits, and once you have applied them, what remains is a genuine three-way trade you must resolve with a decision, not a wish.

## Budgets make the trade explicit

A budget is the triangle written as numbers. For each use case, fix two corners and let the third be the outcome you optimize:

- **Fix cost and latency, optimize quality.** "This interactive feature must answer in under 6s and cost under $0.03 — get me the best quality I can buy inside that box." Common for consumer-facing, high-volume surfaces.
- **Fix quality and latency, optimize cost.** "This must clear a 90% eval score and return in under 10s — now make it as cheap as possible." Common for important interactive features where a quality floor is non-negotiable.
- **Fix quality and cost, optimize latency.** "This must clear the eval bar and stay under $0.05 — make it as fast as you can within that." Common for offline-ish work that still wants to be snappy.

Naming which corner *floats* is the whole discipline. A budget that does not say which dimension is the free variable is not a budget — it is three simultaneous wishes, and under load the system will silently pick one for you, usually badly.

## Resolving the trade per use case

Different surfaces of the *same* system sit at different points in the triangle, and a competent architecture lets them. Forcing one global trade-off across an interactive assistant, a bulk back-fill, and a high-stakes analysis is how you end up overpaying for the bulk job and under-delivering on the high-stakes one.

| Use case | Quality | Latency | Cost | Position & levers |
| --- | --- | --- | --- | --- |
| Interactive assistant (high volume) | "good enough" | tight (p95 < 6s) | tight | small model + cache, capped fan-out, sync lane; quality floors out the trade |
| High-stakes analysis (low volume) | maximal | relaxed (minutes OK) | relaxed | big model, deep loops, refinement passes; value clears any reasonable cost |
| Bulk enrichment / back-fill | "good enough" | none (24h OK) | minimal | small model + **batch lane** (~50% off) + cache; latency is free to spend |
| Eval / regression runs | exact | none | minimal | batch lane; this is the canonical latency-tolerant workload |

The architect's deliverable is this table *for the actual system*, with the budgets filled in and a one-line defense of each row. "Why is the assistant on the small model?" → "It clears the assistant's eval gate at a quarter the cost, and the assistant's budget fixes cost and latency, so quality optimizes *within* the box and the small model is inside it." That sentence is the defense; a design that cannot produce it for each surface is undefended.

## When you cannot fit the box

Sometimes the cheapest viable design still busts the budget. The triangle is telling you something, and there are only a few honest responses:

- **Relax the floating corner's neighbor.** If quality is floored and cost is over, the question goes back to the budget owner: is the latency budget actually firm, or can this move to a (cheaper) batch lane? Is the quality floor really 90%, or was that aspirational?
- **Re-examine the value threshold (Chapter 1).** A task that cannot be done inside any defensible budget may be a task that should not be agentic — or not automated at all. The triangle is one of the places the value threshold comes back to bite.
- **Descope the task.** Narrow the input space, reduce fan-out breadth, drop the optional refinement pass. A smaller task fits a smaller box.

What you must *not* do is quietly bust the budget and hope. An over-budget design that ships without a documented trade-off becomes a production cost incident with no paper trail — the exact failure this whole module exists to prevent.

## Defending the trade-off

The architect's output is not just the chosen point; it is the **defense** of why that point and not its neighbors. A defensible cost–latency–quality decision states, for each use case:

1. **The budget** — the two fixed corners and the floating one, as numbers.
2. **The levers applied** — which of caching / routing / batching, and the precondition each one relies on (Chapter 2).
3. **The evidence** — the eval result (mod-304) that proves the cheaper/faster choice still clears the quality bar, and the cost-model output (Chapter 1) that proves it fits the cost box.
4. **The trade you accepted** — what you gave up, stated plainly ("we accept p95 quality dips on the long tail of multi-hop questions in exchange for halving cost; high-stakes questions route to the big-model lane").

That fourth point is the mark of an architect rather than an optimizer: naming the cost of the decision out loud, so the organization is choosing it with open eyes rather than discovering it in an incident review. A trade-off you can defend in those four lines is a decision; one you cannot is a liability.

## Key takeaways

- Cost, latency, and quality are a **triangle**: the Chapter 2 levers bend it, but past their limits you must resolve a genuine three-way trade with an explicit choice.
- A **budget** fixes two corners and names the third as the free variable; a budget that does not say which dimension floats is three wishes, not a budget.
- Let **different surfaces sit at different points** — interactive, high-stakes, bulk, and eval workloads each get their own row, budget, and lever set; one global trade-off overpays for some and under-delivers for others.
- When no design fits the box, the honest moves are to **relax a budget, revisit the value threshold, or descope** — never to quietly overrun; and every chosen trade ships with a four-line defense (budget, levers, evidence, accepted trade).
