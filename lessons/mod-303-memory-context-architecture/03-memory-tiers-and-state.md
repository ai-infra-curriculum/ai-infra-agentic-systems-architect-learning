# Chapter 3 — Where State Lives: Memory Tiers and Persistence Boundaries

Chapters 1 and 2 were about the *window* — a volatile, per-turn resource. This chapter is about everything that has to outlive the window: the agent's **memory**. The architect's question is deceptively simple and consequential: for each piece of state an agent touches, *where does it live, how long does it last, and who can read it?* Answer it well and the system is coherent and private; answer it by default — "throw it all in one vector store" — and you get an agent that forgets what it should remember, remembers what it should forget, and leaks one user's data into another's session.

The discipline here is **state placement**: classifying agent state into tiers and drawing explicit **persistence boundaries** between them.

## The three memory tiers

Borrowing the cognitive-science framing (and matching how production agent memory systems actually organize themselves):

```text
  ┌─────────────────────────────────────────────────────────────┐
  │  WORKING MEMORY            scope: this turn / this task       │
  │  the context window itself  lifetime: ephemeral (seconds)     │
  │  current plan, recent turns, persistence: none (recomputed)   │
  │  retrieved chunks           → Chapters 1 & 2 govern this      │
  └───────────────────────────────┬─────────────────────────────┘
                                   │ write at session end / checkpoint
                                   ▼
  ┌─────────────────────────────────────────────────────────────┐
  │  EPISODIC MEMORY            scope: this user / this thread    │
  │  past sessions, what was     lifetime: session → weeks        │
  │  done & decided, summaries   persistence: per-user store      │
  │  of prior conversations      → "what happened before"        │
  └───────────────────────────────┬─────────────────────────────┘
                                   │ distill / promote durable facts
                                   ▼
  ┌─────────────────────────────────────────────────────────────┐
  │  LONG-TERM / SEMANTIC MEMORY  scope: user or whole system    │
  │  durable facts, preferences,  lifetime: indefinite            │
  │  learned procedures, the      persistence: durable store + KB │
  │  knowledge base               → "what is true / how to do X"  │
  └─────────────────────────────────────────────────────────────┘
```

- **Working memory** is the context window: the current task, recent turns, the active plan, freshly retrieved content. It is fast, expensive per token, and **ephemeral** — when the session ends, it's gone unless something wrote it down. Everything in Chapters 1–2 is working-memory management.
- **Episodic memory** is "what happened in this user's history": prior sessions, decisions made, threads left open, summaries of past conversations. It is **per-user** (or per-thread) and lives for the lifetime of a relationship — a session, a project, weeks. It answers *"what did we already do / decide?"*
- **Long-term / semantic memory** is durable, distilled knowledge: a user's stable preferences, learned procedures, and the shared knowledge base the agent reasons over. It is **indefinite** and may be per-user (preferences) or system-wide (the knowledge base). It answers *"what is true"* and *"how do we do this."*

The flow between them is the architecture: working memory is **promoted** to episodic at checkpoints and session end; episodic is **distilled** into long-term when a fact proves durable. State that is never promoted is forgotten on purpose; state that is promoted too eagerly pollutes the lower tiers with noise.

## Persistence boundaries are the real design

The tiers are a vocabulary. The *decisions* are the boundaries between them — and each boundary forces three questions an architect must answer explicitly.

### 1. The promotion boundary: what crosses, and when?

Not everything in working memory deserves to persist. A transient calculation, a retrieved chunk, a discarded reasoning path — these die with the window, correctly. What *should* promote is the distilled outcome: decisions, durable facts, open threads, artifacts produced.

Define the promotion rule explicitly. Common patterns:

- **At session end**, write a summary (the decisions, what was produced, what's unresolved) to episodic memory — this is structured note-taking ([Chapter 1](01-context-engineering-strategies.md)) crossing a tier boundary.
- **On an explicit signal** — the user states a preference ("always use metric units"), or the agent completes a milestone — write directly to long-term.
- **On distillation** — periodically compress episodic history into long-term facts, dropping the per-session detail.

The anti-pattern is **promote-everything**: writing every turn to a vector store "so the agent remembers." That doesn't build memory; it builds noise that the agent will later retrieve and rot on (Chapter 2). Memory you don't curate is memory that hurts you.

### 2. The scope boundary: whose memory is this?

This is the boundary that, gotten wrong, becomes a security incident. State is scoped to one of:

- **A single turn** (working memory) — no isolation concern.
- **A single user / thread** (episodic, per-user long-term) — **must** be isolated per user. One user's episodic memory must never surface in another's session. This is a hard multi-tenancy boundary, enforced by a partition key (user ID) on every read and write, not by hoping the retrieval query happens to be specific enough.
- **The whole system** (shared knowledge base) — readable by all, but then it must contain *no* user-private data. The knowledge base is for facts about the world/domain, not facts about users.

The architectural rule: **draw the scope key into the storage schema, not into the query.** A retrieval that filters by user ID in application code is one missing `WHERE` clause from a cross-tenant leak. A store partitioned by user ID at the namespace level cannot leak across tenants even if the query is wrong. Privacy here is a placement decision, and it ties directly to [mod-306](../mod-306-guardrails-safety-security/README.md) and [mod-309](../mod-309-governance-compliance-domain/README.md).

### 3. The consistency boundary: how fresh, how durable?

Each tier has different consistency needs, and conflating them is a common mistake:

- **Working memory** needs no durability — it's recomputed each session. It needs to be *fresh*: if it caches a value that changed (JIT retrieval, Chapter 1), it must re-fetch.
- **Episodic memory** needs **session durability** and **read-your-writes** within a user's thread — if a user told the agent something this session, the agent must see it next session. Eventual consistency across regions is usually fine; staleness within a thread is not.
- **Long-term memory** needs **durability** and benefits from **versioning** — preferences change, facts get corrected, procedures get superseded. A long-term store that can't represent "this fact replaced that one" forces the agent to reason over contradictions. Treat updates as new versions, not overwrites, where audit matters.

```text
  tier            durability     consistency need        update model
  ─────────────── ────────────── ─────────────────────── ─────────────────
  working         none           fresh (re-fetch stale)  recomputed
  episodic        per-session    read-your-writes/thread  append summaries
  long-term       indefinite     durable + auditable     versioned upsert
```

## A worked placement

A customer-support agent. Walk every piece of state to a tier and a boundary:

| State | Tier | Scope | Persistence rule |
| --- | --- | --- | --- |
| Current ticket conversation | working | turn/task | none — in the window |
| Retrieved KB articles this turn | working | turn | JIT, dropped after use |
| Summary of this ticket's resolution | episodic | per-user | written at ticket close |
| "Customer prefers email over phone" | long-term | per-user | written on signal, versioned |
| Product documentation / policies | long-term | system | shared KB, no user data, RAG ([Chapter 4](04-rag-and-knowledge-stores.md)) |
| The agent's tool list & system prompt | working (stable) | system | front of window, cached |

The leak you're preventing: the customer's resolution summary (episodic, per-user) must *never* land in the shared product-documentation store (system). The fact that both are "things the agent remembers" is exactly the trap — they belong on opposite sides of the scope boundary.

## Starter guidance — the memory-tier placement diagram

For any agent, produce this. It's the artifact you defend in [exercise-02](exercises/exercise-02-memory-tier-placement-design.md).

```text
MEMORY-TIER PLACEMENT
Agent: ____________________

For EACH piece of state the agent reads or writes, fill one row:

State item        Tier        Scope key      Lifetime    Store           Promotion rule
───────────────── ─────────── ────────────── ─────────── ─────────────── ──────────────────────
________________  working/    turn/user/sys  ephemeral/  none/per-user/  none / at session end /
                  episodic/                   session/    shared store    on signal / on distill
                  long-term                   indefinite

BOUNDARY CHECKS (answer for the whole table):
  □ Promotion:    Is every persisted item one you can justify keeping? (no promote-everything)
  □ Scope:        Is every per-user item partitioned by a key in the SCHEMA, not the query?
  □ Privacy:      Does the system-scoped store contain ZERO user-private data?
  □ Consistency:  Does each tier's store meet its freshness/durability need (table above)?
  □ Forgetting:   Is there a deletion path for per-user memory (a user can be forgotten)?
```

The last check is the one teams skip and regulators don't: a memory architecture must have a **forgetting path**. If a user's data lives across working (gone automatically), episodic, and long-term tiers, "delete this user" has to reach every per-user partition. Designing the scope key in from the start is what makes that one operation instead of an archaeology project.

## Key takeaways

- Agent state lives in three tiers — **working** (the window, ephemeral), **episodic** (per-user history, session-to-weeks), and **long-term/semantic** (durable facts, preferences, the knowledge base) — connected by *promotion* (working→episodic) and *distillation* (episodic→long-term).
- The architecture is in the **boundaries**, not the tiers: the **promotion** boundary (what persists, and never promote-everything), the **scope** boundary (per-turn vs. per-user vs. system — partition by a schema key, not a query), and the **consistency** boundary (working=fresh, episodic=read-your-writes, long-term=durable+versioned).
- The scope boundary is a **security boundary**: per-user memory must be isolated by a partition key in the schema; system-scoped stores must hold zero user-private data. Design a **forgetting path** from day one.
- The deliverable is a **placement diagram**: every state item mapped to a tier, scope, lifetime, store, and promotion rule, with the boundary checks answered.
