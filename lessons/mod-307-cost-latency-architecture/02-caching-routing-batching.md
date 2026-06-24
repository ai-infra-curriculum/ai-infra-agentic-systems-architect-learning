# Chapter 2 — Architectural Levers: Caching, Routing, and Batching

Chapter 1 gave you the cost model. This chapter gives you the three levers that move it — and the discipline to treat them as **architecture**, not as last-minute tuning. Each lever has a quantified multiplier and a precondition. An architect who knows the multipliers can redraw a viability line on a whiteboard; an engineer who finds out about them after launch is rewriting the system.

The levers, in the order you should consider them:

1. **Prefix caching** — pay full price for stable context once, near-nothing on reuse.
2. **Model routing** — send each call to the cheapest model that can do *that* call.
3. **Batch processing** — trade latency for a flat discount on work that is not time-sensitive.

## Lever 1: Prefix / prompt caching

Recall the cost driver from Chapter 1: in an agent loop, the **stable prefix** — system prompt, tool definitions, retrieved context, examples — is re-sent on every turn and paid for every time. Prefix caching breaks that. You mark a stable span as cacheable; the provider stores the computed prefix, and subsequent calls that share that exact prefix read it back at a steep discount instead of reprocessing it.

The economics, using illustrative round multipliers (verify current rates in [resources.md](resources.md)):

- **Cache write** (first call that establishes the entry): a premium over base input, often around **1.25×** the normal input price.
- **Cache read** (every subsequent hit): a deep discount, often around **0.1×** (a 90% saving) of the normal input price.
- Entries are **ephemeral** with a short time-to-live (on the order of minutes), refreshed on each hit.

The lever pays off whenever a large stable prefix is reused more than a couple of times before it goes cold — which is exactly the shape of an agent loop and of fan-out workers that share a common system prompt and toolset. The design rules:

- **Order the prompt cache-first.** Put everything stable — system prompt, tools, long-lived retrieved context — *at the front*, and the volatile per-turn content *after* it. The cache matches on a prefix; one early-changing token invalidates everything downstream of it. Prompt *layout* is now a cost decision.
- **Cache the shared worker preamble.** When ten workers share a system prompt and tool schema, that preamble is written once and read nine times. Fan-out is where caching pays the most, because the same prefix is reused across siblings.
- **Watch the TTL against your traffic.** If requests arrive slower than the cache expires, you pay the write premium repeatedly and never collect the read discount. Caching helps warm paths; it can mildly *hurt* cold, low-frequency ones.

In the SDK, the lever is a `cache_control` marker on the stable span, and the payoff shows up in the usage fields:

```python
import anthropic

client = anthropic.Anthropic()

resp = client.messages.create(
    model="claude-sonnet-4-0",
    max_tokens=1024,
    system=[
        # Stable prefix marked cacheable: written once, read on every reuse.
        {"type": "text", "text": LARGE_STABLE_CONTEXT,
         "cache_control": {"type": "ephemeral"}},
    ],
    messages=[{"role": "user", "content": volatile_question}],
)

u = resp.usage
# These fields are how you VERIFY the lever is actually firing in production.
print("cache_creation_input_tokens:", u.cache_creation_input_tokens)  # write
print("cache_read_input_tokens:    ", u.cache_read_input_tokens)      # read (cheap)
print("input_tokens (uncached):    ", u.input_tokens)
```

> **Measure the hit rate, do not assume it.** A cache you designed for but never confirmed is a line in a slide deck. `cache_read_input_tokens` divided by total prefix tokens is your real hit rate, and it belongs on the dashboard (mod-305) next to cost.

## Lever 2: Model routing

Not every call in an agentic system needs the frontier model. Decomposition and synthesis may need strong reasoning; a worker that extracts fields from a document, classifies an intent, or formats an answer often does not. **Model routing** sends each call to the cheapest model that clears that call's quality bar — turning "one model for everything" into a tiered fleet.

The multiplier is large because the price gap between tiers is large. If a small model is roughly **4×–5× cheaper** per token than a frontier model (illustrative; see pricing in [resources.md](resources.md)), moving the high-volume, low-difficulty calls down a tier can cut total cost by a third or more *with no quality loss on those calls* — because those calls never needed the big model.

There are two routing styles, and an architect should know when each fits:

- **Static routing (by role).** The architecture assigns models to roles at design time: orchestrator → big model, extraction workers → small model, formatter → small model. Predictable, cheap to reason about, no routing call to pay for. This is the default; reach for it first.
- **Dynamic routing (by difficulty).** A lightweight classifier (a cheap model or a heuristic) inspects each request and routes hard ones up, easy ones down. More adaptive, but you pay for the classifier on every request and you inherit a new failure mode: a misroute that sends a hard task to a model that fails it. Reserve dynamic routing for genuinely mixed input streams where static role-assignment leaves money on the table.

```text
                         ┌────────────────────┐
        request ────────▶│   router / gate    │
                         │ (static by role,   │
                         │  or cheap classifier)
                         └─────────┬──────────┘
                  hard / reasoning │ easy / bulk
                          ┌────────┴─────────┐
                          ▼                  ▼
                  ┌───────────────┐   ┌───────────────┐
                  │  BIG model    │   │  SMALL model  │
                  │ orchestrate,  │   │ extract,      │
                  │ synthesize    │   │ classify,     │
                  │               │   │ format        │
                  └───────────────┘   └───────────────┘
```

The design discipline: **route on the call, not the system.** "We use the big model" is not an architecture decision; "decomposition and synthesis use the big model, the six extraction workers use the small one" is. Every call you can defensibly downshift is a permanent line-item cut. The risk to manage is **silent quality regression** — a downshift that looks fine in a demo and degrades on the long tail. Routing decisions belong in the eval harness (mod-304): you only downshift a call after the cheaper model has cleared that call's quality gate on a real test set.

## Lever 3: Batch processing

The first two levers cut the price of a call. Batching changes *when* the call runs in exchange for a discount. Providers offer an asynchronous **batch** mode — submit a large set of independent requests, get results within a window (commonly up to 24 hours) — at roughly **50% off** standard pricing (verify in [resources.md](resources.md)).

The precondition is strict and binary: the work must be **latency-tolerant**. Batching is wrong for anything a user is waiting on. It is right — and frequently *underused* — for the large, asynchronous workloads that hide in agentic systems:

- Nightly summarization, enrichment, or re-indexing of a corpus.
- Bulk evaluation runs against a test set (mod-304) — eval traffic is the canonical batch workload.
- Offline classification, tagging, or back-fill jobs.
- Pre-computing artifacts (embeddings, digests, derived context) that the live system will later read.

```python
import time
import anthropic

client = anthropic.Anthropic()

# Independent, non-time-sensitive requests → 50%-off async lane.
batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": f"doc-{i}",
            "params": {
                "model": "claude-3-5-haiku-latest",  # cheap model + batch = both levers
                "max_tokens": 512,
                "messages": [{"role": "user", "content": f"Summarize:\n{doc}"}],
            },
        }
        for i, doc in enumerate(corpus)
    ]
)

while client.messages.batches.retrieve(batch.id).processing_status != "ended":
    time.sleep(30)  # offline job — polling latency is acceptable by definition

for r in client.messages.batches.results(batch.id):
    store(r.custom_id, r.result.message.content[0].text)
```

The architectural move is to **split the workload by latency requirement** at design time: a synchronous lane for what the user waits on, an asynchronous batch lane for everything else. Teams that run one synchronous lane for all traffic are paying full price for work no one is waiting on.

## Stacking the levers

The levers compose, and an architect plans the stack, not the individual tweak. The biggest savings come from applying all three where each one's precondition holds:

```text
  base cost
     │  route bulk calls to a small model        →  ~ -30% to -50% of total
     │  cache the shared stable prefix           →  ~ -90% on cached input tokens
     │  batch the latency-tolerant lane          →  ~ -50% on that lane
     ▼
  designed cost  (a multiple cheaper than "big model, no cache, all sync")
```

Order of operations when you sit down to design:

1. **Route first.** Pick the cheapest viable model per call — it scales every downstream number.
2. **Cache second.** Lay out the prompt cache-first and confirm the read hit rate.
3. **Batch last.** Move every latency-tolerant call into the async lane.

Each lever has a precondition you must state explicitly in the design — caching needs prefix reuse before TTL, routing needs the cheap model to clear the call's eval gate, batching needs latency tolerance. A lever applied where its precondition fails is not a saving; it is a regression (a cold cache that only pays write premiums, a downshift that fails the long tail, a batched call a user is blocked on). The art is matching each lever to the calls that satisfy its precondition — and writing that match down where the next architect can audit it.

## Key takeaways

- **Prefix caching**: order the prompt cache-first, mark the stable span, and reuse it across loop turns and fan-out siblings — roughly a 90% read discount, but only above the cache TTL's frequency. Confirm the hit rate from `cache_read_input_tokens`.
- **Model routing**: assign the cheapest model that clears each call's quality gate; static-by-role first, dynamic-by-difficulty only for mixed streams. A downshift is permanent savings — gated on the eval harness so quality does not silently regress.
- **Batch processing**: move every latency-tolerant call (eval runs, nightly jobs, back-fills) to the ~50%-off async lane; split synchronous and asynchronous lanes at design time.
- The levers **stack** (route → cache → batch), but each has a **precondition** you must state in the design; a lever applied where its precondition fails is a regression, not a saving.
