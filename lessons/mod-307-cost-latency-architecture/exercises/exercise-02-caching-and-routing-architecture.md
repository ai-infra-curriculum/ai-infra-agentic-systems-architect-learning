# exercise-02: Caching and Routing Architecture

**Estimated effort:** 3 hours

## Objective

Take the cost model from exercise-01 and **cut it** by designing the three architectural levers — prefix caching, model routing, and batch processing — into the system. You will quantify each lever's multiplier against your own baseline, produce a routed-architecture diagram, and write the cache layout and routing table as design artifacts an implementing engineer could build from.

## Background

This exercise covers material from:

- [Chapter 2 — Architectural Levers: Caching, Routing, and Batching](../02-caching-routing-batching.md)
- [Chapter 1 — Token Economics & Value Thresholds](../01-token-economics.md) (your baseline)

You design the levers; you do not have to ship them. A few real API calls to **confirm the cache fires** (Task 2) are encouraged but optional — the usage fields make the check cheap.

## Prerequisites

- A completed exercise-01 baseline cost model (you will apply levers to *those* numbers).
- The orchestrator-worker fan-out shape from [mod-302](../../mod-302-multi-agent-orchestration/README.md).

## Tasks

### 1. Lever inventory: where does each one apply?

- Walk every call in your exercise-01 system and tag it with the levers that apply and their **preconditions**:
  - **Cache** — does this call share a large stable prefix with other calls, reused before TTL? (Fan-out siblings sharing a preamble are the prime case.)
  - **Route** — can this call run on the cheaper model without failing its job? (Extraction/format workers usually can; orchestration usually cannot.)
  - **Batch** — is this call latency-tolerant? (Anything a user waits on is not.)
- Produce a table: one row per call, columns `cache? / route-to / batch?`, with a one-line precondition justification each.

### 2. Design and confirm the cache layout

- Lay out the prompt **cache-first**: specify exactly which span (system prompt, tool schema, shared retrieved context) is marked cacheable, and confirm the volatile per-turn content sits *after* it.
- Compute the cache multiplier on your baseline: pick the worker preamble, count its tokens, and compute write cost (≈1.25× input) once vs. read cost (≈0.1× input) across the reuse count (loop turns × fan-out siblings). Report the saving on cached input tokens.
- **Optional confirmation:** make two real calls sharing a cached prefix and report `cache_creation_input_tokens` and `cache_read_input_tokens` from the usage object. A cache you did not confirm is a slide, not a saving.

```python
# Optional: confirm the cache actually fires.
import anthropic
client = anthropic.Anthropic()
for i in range(2):  # call 1 writes the entry, call 2 should read it
    r = client.messages.create(
        model="claude-3-5-haiku-latest", max_tokens=64,
        system=[{"type": "text", "text": SHARED_PREAMBLE,
                 "cache_control": {"type": "ephemeral"}}],
        messages=[{"role": "user", "content": f"ping {i}"}],
    )
    u = r.usage
    print(i, "write:", u.cache_creation_input_tokens, "read:", u.cache_read_input_tokens)
```

### 3. Design the routing table

- Decide **static (by role)** vs. **dynamic (by difficulty)** for this system, and justify the choice (default to static; reach for dynamic only on genuinely mixed input).
- Produce the routing table: each role/call → assigned model → the eval gate that must pass before the downshift is allowed (Chapter 2 ties routing to mod-304).
- Compute the routing multiplier: re-price the system with bulk calls downshifted to the small model and report the percentage cut vs. baseline.

### 4. Design the batch lane

- Identify which workloads in (or adjacent to) the system are latency-tolerant and belong in the ~50%-off async lane (e.g., nightly enrichment, eval runs, pre-computed context).
- For each, state the latency budget that makes batching safe ("results within 24h is acceptable because …").
- Compute the batch multiplier on that lane.

### 5. Stack the levers and diagram

- Apply the levers in order — **route → cache → batch** — and report the **stacked** cost vs. the exercise-01 baseline (a single before/after number plus the per-lever contributions).
- Produce a **routing-architecture diagram** (ASCII or image) showing the router, the big/small tiers, the shared cached prefix across fan-out siblings, the meter, and the sync/batch lane split.

## Starter guidance

A routing-architecture diagram template to fill in:

```text
                         ┌────────────────────┐
        request ────────▶│  router / gate     │   static-by-role OR
                         │  (you choose)      │   cheap classifier
                         └─────────┬──────────┘
                  hard / reasoning │ easy / bulk
                          ┌────────┴─────────┐
                          ▼                  ▼
                  ┌───────────────┐   ┌───────────────┐
                  │  BIG model    │   │  SMALL model  │
                  │ orchestrate / │   │ extract /     │
                  │ synthesize    │   │ classify /    │
                  │               │   │ format        │
                  └──────┬────────┘   └───────┬───────┘
                         │  fan-out workers   │
                         ▼ share ONE cached prefix
                  ┌──────────────────────────────────┐
                  │  [cached preamble] + [volatile]   │  ← cache-first layout
                  └──────────────────────────────────┘
                         │ meter every call
                         ▼
                  ┌──────────────────────────────────┐
                  │  ASYNC / BATCH LANE (~50% off)    │  ← latency-tolerant only
                  └──────────────────────────────────┘
```

Per-lever multiplier worksheet:

```python
"""exercise-02 lever multipliers, applied to the exercise-01 baseline."""
BASELINE = 0.0  # paste your exercise-01 per-task cost here

CACHE_READ_FACTOR = 0.10   # cached input ≈ 10% of normal input price
CACHE_WRITE_FACTOR = 1.25  # first write ≈ 125% of normal input price
SMALL_VS_BIG = 0.25        # small model ≈ 1/4 the per-token cost (illustrative)
BATCH_FACTOR = 0.50        # batch lane ≈ 50% of standard price

# Fill these from your tagged call inventory (Task 1):
cached_input_tokens = 0     # prefix tokens eligible for cache reads
reuses = 0                  # loop turns * fan-out siblings sharing the prefix
bulk_call_cost = 0.0        # baseline $ of calls you can downshift
batchable_call_cost = 0.0   # baseline $ of latency-tolerant calls

# ... compute saved $ per lever, then the stacked total. Show your work.
print("report per-lever savings and the stacked before/after here")
```

Worked diagrams, the filled routing table, and the confirmed cache numbers live in the [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

## Deliverables

1. `levers.md` — the tagged call inventory, the cache layout, the routing table (with eval gates), and the batch-lane spec.
2. `routing-architecture.{txt,png}` — the diagram.
3. `multipliers.{py,xlsx}` — per-lever and stacked before/after numbers against the exercise-01 baseline.

## Acceptance criteria

You can demonstrate that:

- Every call is tagged with applicable levers **and their preconditions**; no lever is applied where its precondition fails.
- The cache layout is **cache-first**, names the exact cacheable span, and reports the read-discount saving (with a confirmed `cache_read_input_tokens` if you ran the optional check).
- The routing table assigns the cheapest viable model per call and ties each downshift to an **eval gate**, with a static-vs-dynamic choice justified.
- A batch lane is specified with a stated latency budget per workload and a computed discount.
- A stacked **before/after** cost is reported against the exercise-01 baseline, with per-lever contributions.
- A routing-architecture diagram shows router, tiers, shared cached prefix, meter, and sync/batch split.

## Reflection

In `NOTES.md`:

1. Which lever delivered the biggest cut for this system — and would that ranking hold if traffic were 10× more bursty (cold caches)?
2. Name a call where you were tempted to downshift but the eval gate would (or did) fail. How did you decide?
3. Where did a precondition *fail* — a cache that would mostly pay write premiums, or a "batchable" job a user actually waits on? What did you do instead?

## Stretch goals

- Add **dynamic routing** with a cheap classifier in front: price the classifier overhead and the cost of a misroute, then decide whether it beats static-by-role for this system.
- Model the cache under **bursty traffic**: at what request inter-arrival time does the TTL stop paying off? Find the break-even frequency.
- Combine **small model + batch** on the enrichment lane and report the doubled-up multiplier.
