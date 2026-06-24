# Quiz 1 — Cost & Latency Architecture

Knowledge check for [mod-307](../README.md). Answers are at the bottom; try each question before scrolling. Covers all four chapters.

## Questions

### 1. What cost scales with

In a multi-agent system, the total cost scales most directly with:

- A. The number of user requests.
- B. The number of model calls — fan-out and re-sent history turn one request into many calls.
- C. The number of tenants, regardless of usage.
- D. The size of the vector index.

### 2. Input vs. output

Why can't you reason about per-call cost by looking at the input token count alone?

- A. Input tokens are free.
- B. Output is priced several times higher per token, so a much smaller output column can still drive a large share of the line cost.
- C. Output tokens are always more numerous than input tokens.
- D. Providers bill input and output at the same rate.

### 3. Re-sent history

Why does a 5-turn agent loop over a 10,000-token context cost far more than 10,000 input tokens?

- A. The model charges a flat per-loop surcharge.
- B. The growing transcript is re-sent every turn, so the same context tokens are paid for repeatedly across turns.
- C. Output tokens are counted as input after turn one.
- D. It doesn't — it costs exactly 10,000 input tokens.

### 4. The fan-out multiplier

Anthropic reported that their multi-agent research system used roughly how much more token volume than a single chat interaction?

- A. About the same.
- B. Roughly 2×.
- C. Roughly 15×.
- D. Roughly 100×.

### 5. The value threshold

The value-threshold test says to build the agent only when:

- A. The model is capable enough to do the task.
- B. `value_per_task > cost_per_task × safety_margin`.
- C. The task cannot be enumerated as steps.
- D. A competitor has already shipped one.

### 6. The trap quadrant

A task with **high cost per task** and **low value per task** should be:

- A. Built as an agent and optimized later.
- B. Not built as an agent — it is a workflow, a cached single call, a cheaper model, or not automated.
- C. Always built as a workflow with a big model.
- D. Built but rate-limited.

### 7. The p95 tenant

Why model a p95 tenant and not just the mean when rolling up cost?

- A. The mean is always wrong.
- B. The mean hides the heavy tenant who can run a negative-margin account on a flat price.
- C. p95 is required by the Batches API.
- D. Mean and p95 are identical for token cost.

### 8. Cache-first layout

Why must the stable, cacheable span sit at the **front** of the prompt?

- A. The model reads front-to-back and ignores the end.
- B. Caching matches on a prefix; one early-changing token invalidates everything downstream of it.
- C. The front of the prompt is billed at a lower rate.
- D. It has no effect; cache_control works anywhere.

### 9. Cache TTL trap

When does prefix caching *hurt* rather than help?

- A. When the prefix is very large.
- B. When requests arrive slower than the cache TTL — you keep paying the write premium and never collect the read discount.
- C. When using the small model.
- D. Caching never hurts.

### 10. Routing the call, not the system

The architectural discipline for model routing is:

- A. "Use the big model" is the decision.
- B. Route on the *call* — assign the cheapest model that clears each call's quality gate (e.g., orchestrate on the big model, extract on the small one).
- C. Always use dynamic routing with a classifier.
- D. Route by tenant tier only.

### 11. Gating a downshift

Before downshifting a call to a cheaper model, an architect requires:

- A. Nothing — cheaper is always better.
- B. The cheaper model to clear that call's quality gate on a real test set (mod-304), to avoid silent regression.
- C. Approval from the CFO.
- D. A larger context window.

### 12. The batch precondition

The Batches API's ~50% discount is available only when:

- A. The requests use the big model.
- B. The work is latency-tolerant — nothing a user is actively waiting on.
- C. There are fewer than 100 requests.
- D. Caching is also enabled.

### 13. Lever order

The recommended order for applying the three levers at design time is:

- A. Batch, then cache, then route.
- B. Route (cheapest viable model per call) → cache (cache-first layout) → batch (latency-tolerant lane).
- C. Cache everything first, then decide models.
- D. The order doesn't matter.

### 14. Caps as budget

Why must every fan-out and loop carry a hard cap (max workers N, max turns T)?

- A. To make the diagram look complete.
- B. Re-sent history and unbounded fan-out make an uncapped loop or decomposition a runaway-cost incident; "the model will probably stop" is not a budget.
- C. Caps improve answer quality.
- D. Providers reject uncapped requests.

### 15. The control plane

Which set of components distinguishes a cost-*aware* reference architecture from a cost-*blind* one?

- A. A bigger model and more workers.
- B. A token meter, a budget guard, and a kill switch reading that meter.
- C. A vector store and a reranker.
- D. A streaming response and a CDN.

### 16. Graceful degradation

When a task approaches its cost budget, graceful degradation means:

- A. Immediately failing the request with an error.
- B. Downshifting to a cheaper model, skipping the optional refinement pass, or returning the best partial answer flagged as truncated.
- C. Doubling the fan-out to finish faster.
- D. Silently ignoring the budget.

### 17. Cost as an SLO

Why budget and alert on **p95/p99** cost-per-task rather than the mean?

- A. The mean is harder to compute.
- B. The tail — tasks that fan out wide and loop deep — is what generates the surprise bill and the SLA breach; the mean hides it.
- C. p99 is cheaper to monitor.
- D. SLOs legally require percentiles.

### 18. The triangle

Cost, latency, and quality form a triangle because:

- A. You can always maximize all three with enough engineering.
- B. Improving one generally trades against another past the levers' limits; every design is a point inside, never a corner.
- C. They are independent and never interact.
- D. Latency is a function of quality only.

### 19. What a budget must name

A budget that does not say **which dimension floats** is:

- A. Fine, as long as the numbers are right.
- B. Not a budget — it is three simultaneous wishes, and under load the system picks one for you, usually badly.
- C. Only a problem for interactive surfaces.
- D. Acceptable if quality is high.

### 20. When the box won't fit

The cheapest viable design still busts the budget. The honest architect's responses are:

- A. Ship it and hope traffic stays low.
- B. Relax the floating corner's neighbor, revisit the value threshold, or descope — never quietly overrun.
- C. Remove all caps so it runs faster.
- D. Switch every call to the big model.

## Answer key

1. **B** — Cost scales with model calls; fan-out and re-sent history turn one request into many ([Chapter 1](../01-token-economics.md)).
2. **B** — Output is priced several times higher, so a small output column can still drive a large share of cost ([Chapter 1](../01-token-economics.md)).
3. **B** — The growing transcript is re-sent every turn, so context tokens are paid for repeatedly ([Chapter 1](../01-token-economics.md)).
4. **C** — Anthropic reported roughly 15× the token use of a single chat interaction ([Chapter 1](../01-token-economics.md)).
5. **B** — Build only when `value_per_task > cost_per_task × safety_margin` ([Chapter 1](../01-token-economics.md)).
6. **B** — The trap quadrant: real cost, thin value — use a workflow, a cached call, a cheaper model, or no automation ([Chapter 1](../01-token-economics.md)).
7. **B** — The mean hides the heavy tenant who can run a negative-margin account on a flat price ([Chapter 1](../01-token-economics.md)).
8. **B** — Caching matches on a prefix; one early-changing token invalidates everything downstream ([Chapter 2](../02-caching-routing-batching.md)).
9. **B** — If requests arrive slower than the TTL, you pay write premiums and never collect read discounts ([Chapter 2](../02-caching-routing-batching.md)).
10. **B** — Route on the call: cheapest model that clears each call's quality gate ([Chapter 2](../02-caching-routing-batching.md)).
11. **B** — The cheaper model must clear the call's eval gate (mod-304) to prevent silent regression ([Chapter 2](../02-caching-routing-batching.md)).
12. **B** — The batch discount requires latency tolerance — nothing a user waits on ([Chapter 2](../02-caching-routing-batching.md)).
13. **B** — Route → cache → batch ([Chapter 2](../02-caching-routing-batching.md)).
14. **B** — Uncapped fan-out and loops are runaway-cost incidents; every one needs a hard cap ([Chapter 3](../03-cost-models-and-slas.md)).
15. **B** — The control plane: token meter, budget guard, kill switch ([Chapter 3](../03-cost-models-and-slas.md)).
16. **B** — Degrade by downshifting, skipping optional passes, or returning a flagged partial answer ([Chapter 3](../03-cost-models-and-slas.md)).
17. **B** — The tail generates the surprise bill and SLA breach; the mean hides it ([Chapter 3](../03-cost-models-and-slas.md)).
18. **B** — The levers bend the triangle but have limits; every design is a point inside, never a corner ([Chapter 4](../04-cost-latency-quality-budgets.md)).
19. **B** — A budget that doesn't name the floating dimension is three wishes; the system picks one under load ([Chapter 4](../04-cost-latency-quality-budgets.md)).
20. **B** — Relax a neighbor, revisit the value threshold, or descope — never quietly overrun ([Chapter 4](../04-cost-latency-quality-budgets.md)).
