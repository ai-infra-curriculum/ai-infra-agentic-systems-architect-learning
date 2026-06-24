# exercise-02: Memory-Tier Placement Design

**Estimated effort:** 3 hours

## Objective

Take a real agent and place **every** piece of state it touches into a memory tier, then defend the persistence boundaries between tiers — promotion, scope, and consistency — including a forgetting path. This is a design exercise: the deliverable is a **placement diagram** and a short defense, the kind of artifact you'd hand an engineering team and a privacy reviewer before they build the memory layer.

## Background

This exercise covers material from:

- [Chapter 3 — Where State Lives: Memory Tiers and Persistence Boundaries](../03-memory-tiers-and-state.md)
- [Chapter 1 — Context Engineering as a Token Budget](../01-context-engineering-strategies.md) (working memory = the window)

No code is required to pass, though a thin prototype is a stretch goal. The skill being assessed is *placement and boundary reasoning*.

## Prerequisites

- A concrete agent to design for. Use one of the provided scenarios or your own:
  - **Customer-support agent** — handles tickets, learns customer preferences, reasons over product docs.
  - **Personal coding assistant** — works across sessions on one user's repos, remembers project conventions.
  - **Multi-tenant research assistant** — many users, shared public knowledge base, private per-user notes.
- Familiarity with the three tiers (working / episodic / long-term) and the three boundaries (promotion / scope / consistency).

## Tasks

### 1. Enumerate the state

List **every** piece of state your agent reads or writes. Be exhaustive — system prompt, tool list, the current conversation, retrieved chunks, user preferences, prior-session summaries, the shared knowledge base, any IDs or credentials. If you can't name it, you can't place it.

### 2. Place each item into a tier

For every item, assign a **tier** (working / episodic / long-term), a **scope key** (turn / per-user / system), a **lifetime**, a **store**, and a **promotion rule** (none / at session end / on signal / on distillation). Use the [Chapter 3 placement table](../03-memory-tiers-and-state.md#starter-guidance--the-memory-tier-placement-diagram).

### 3. Defend the boundaries

Answer the five boundary checks from the chapter for your whole table:

- **Promotion** — is every persisted item justified? Point to anything you deliberately *don't* persist and say why (resist promote-everything).
- **Scope** — is every per-user item partitioned by a key in the **schema**, not the query? Show where the partition key lives.
- **Privacy** — does your system-scoped store contain **zero** user-private data? If a per-user fact and a shared fact look similar, show they're on opposite sides of the boundary.
- **Consistency** — does each tier's store meet its freshness/durability need (working=fresh, episodic=read-your-writes, long-term=durable+versioned)?
- **Forgetting** — trace the operation "delete user U." Show it reaches every per-user partition across episodic and long-term tiers (and any retrieval index).

### 4. Draw the promotion/distillation flow

Produce a small diagram showing what crosses **working → episodic** (at what trigger) and **episodic → long-term** (at what trigger). This is the dynamic view that the static placement table doesn't show.

### 5. Find the leak

State one concrete way your design could leak one user's memory into another user's session if a boundary were drawn wrong, and show the specific mechanism in your design that prevents it. (If you can't find a plausible leak, your enumeration in Task 1 is probably incomplete.)

## Starter guidance

```text
MEMORY-TIER PLACEMENT — Customer-support agent (worked partial)

State item              Tier       Scope   Lifetime    Store          Promotion rule
─────────────────────── ────────── ─────── ─────────── ────────────── ──────────────────────
current ticket convo    working    turn    ephemeral   none (window)  none
retrieved KB articles   working    turn    ephemeral   none (JIT)     none — drop after use
this ticket's summary   episodic   user    weeks       per-user KV    write at ticket close
"prefers email contact" long-term  user    indefinite  per-user store on signal, VERSIONED
product policy docs      long-term  system  indefinite  shared vector  curated; NO user data
─────────────────────── ────────── ─────── ─────────── ────────────── ──────────────────────

PROMOTION / DISTILLATION FLOW
  working ──(at ticket close: write resolution summary)──▶ episodic[user]
  episodic ──(stable preference observed N times)────────▶ long-term[user]
  (product docs enter long-term[system] via a curation pipeline, never from a session)

SCOPE KEY: every episodic / long-term[user] row is namespaced by user_id at the
           store level — a query that forgets the filter still cannot cross tenants.

FORGETTING "delete user U": drop episodic[U] partition + long-term[user][U] rows
           + re-index any per-user retrieval namespace for U. (system store untouched.)
```

## Acceptance criteria

You can demonstrate that:

- Your **placement table** covers every state item the agent touches, each with a tier, scope key, lifetime, store, and promotion rule.
- You **answered all five boundary checks** with specifics, not assertions — especially showing the scope key lives in the schema.
- You have a **promotion/distillation flow** diagram with named triggers.
- You identified a **plausible cross-user leak** and the mechanism that prevents it.
- Your design has a **forgetting path** that provably reaches every per-user partition.

## Reflection

In `NOTES.md`:

1. Which item was hardest to place, and what made the tier ambiguous? How did you resolve it?
2. Where were you tempted to "promote everything," and what would the cost have been (in rot or privacy)?
3. If this agent went multi-region, which consistency boundary would bite first, and how would you adjust?

## Stretch goals

- Prototype the scope boundary: a tiny per-user store (dict-of-dicts keyed by `user_id`) and a retrieval function that *cannot* return another user's data even when called with a wrong query. Write a test that proves it.
- Add a **deletion test**: insert data for two users, delete one, assert the other survives and the deleted user is gone from every tier.
- Extend the design with a **memory-conflict policy**: when long-term memory says "prefers email" but the user just said "call me," which wins, and how is the contradiction represented (versioning)?
