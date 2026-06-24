# Chapter 3 — Workflow & Toolchain Integration

A platform is only as useful as the systems it can reach. An internal agent that cannot open a change in version control, read an issue tracker, consult a knowledge base, or react to a CI result is a chatbot. So the integration layer — how the agent connects to the org's existing toolchains — is where a large fraction of platform engineering effort and platform risk lives. The architect's job here is **interface selection**: every external system exposes more than one way to talk to it, and choosing the wrong one produces integrations that are slow, brittle, rate-limited, or wasteful. This chapter is vendor-neutral on purpose — the four toolchain *categories* and the three *interface styles* recur no matter which specific products an org runs.

## The toolchain categories (vendor-neutral)

Whatever the brand names, internal platforms integrate with four recurring categories of system:

```text
   ┌──────────────────────────────────────────────────────────────┐
   │ VERSION CONTROL        e.g. GitHub, GitLab, Bitbucket          │
   │   open/update changes, read diffs, comment, read history       │
   ├──────────────────────────────────────────────────────────────┤
   │ ISSUE / WORK TRACKER   e.g. Jira, Linear, GitHub Issues        │
   │   read/transition tickets, comment, link work                  │
   ├──────────────────────────────────────────────────────────────┤
   │ KNOWLEDGE BASE         e.g. Confluence, Notion, a wiki, docs    │
   │   read specs, runbooks, design docs for context                │
   ├──────────────────────────────────────────────────────────────┤
   │ CI / CD                e.g. GitHub Actions, GitLab CI, Jenkins  │
   │   trigger builds, read results, react to completion            │
   └──────────────────────────────────────────────────────────────┘
```

The category determines the *shape* of the integration regardless of vendor: a tracker is read/transition/comment; a knowledge base is read-for-context; CI/CD is trigger-and-react. Design to the category, then bind to the specific product — usually behind an **MCP server** ([Chapter 1](01-extension-model-across-platforms.md)) so the binding stays portable and the credential stays out of the model's context ([Chapter 2](02-secure-tool-use-and-context-hydration.md)).

## The three interface styles, and when each wins

Almost every toolchain exposes three ways in. They are not interchangeable; each is right for a different access shape.

### REST — shallow reads and discrete writes

A REST API is a set of resource endpoints. It is the right interface for **discrete writes** (open a change, transition a ticket, post a comment) and **shallow reads** (fetch one resource you can name). It is simple, universally available, and well-understood.

Its weakness is the **N+1 problem**: when you need a thing *and its related things*, REST makes you fetch them one round-trip at a time.

```text
   GET /pulls/42                  → the change
   GET /pulls/42/reviews          → its reviews     (round-trip 2)
   GET /pulls/42/checks           → its checks       (round-trip 3)
   GET /issues/99                 → its linked issue (round-trip 4)
   ...                            → N+1 round-trips, latency stacks up
```

Use REST for: writes, and reads where you want one nameable resource.

### GraphQL — deep, related reads in one round-trip

A GraphQL endpoint lets the client **shape the response**: ask for exactly the fields you need, across related objects, in a single query. The N+1 cascade above collapses to one request:

```text
   query {
     pull(number: 42) {
       title
       reviews(last: 10)   { author state }
       checks              { name conclusion }
       linkedIssue         { number state }
     }
   }
   → one round-trip, exactly the fields needed, nothing more
```

This is also a **context-hydration win** ([Chapter 2](02-secure-tool-use-and-context-hydration.md)): you fetch precisely the slice the task needs, not whole resources you then have to trim. The trade-off: GraphQL endpoints commonly meter by **query cost / point budget** rather than simple request count, so a greedy query can exhaust your budget faster than the equivalent REST calls. Shape queries to the task; do not over-select.

Use GraphQL for: deep, related reads where you would otherwise do an N+1 REST cascade.

### Event-driven (webhooks/events) — react instead of poll

The first two styles are *pull*: the agent asks. But much platform work is **reactive** — act when a change is opened, when a check completes, when a ticket moves. The wrong design polls:

```text
   ANTI-PATTERN (polling)
   ──────────────────────
   loop forever:
     GET /pipeline/123        → "still running"
     sleep 5s                  → wasted calls, rate-limit pressure, latency
     GET /pipeline/123        → "still running"
     ...                       → and you find out late anyway
```

The right design **subscribes**: the toolchain calls *you* when the event happens.

```text
   EVENT-DRIVEN (webhook / event subscription)
   ───────────────────────────────────────────
   CI finishes ──▶ toolchain POSTs event to your endpoint ──▶ enqueue ──▶ agent acts
                   (no polling; you learn immediately; no wasted calls)
```

Use events for: anything where the platform must *react to a change it did not initiate* — CI completion, change opened, review submitted, ticket transitioned.

### The selection rule

```text
   need a discrete write, or one nameable resource?   → REST
   need deep/related data in one round-trip?           → GraphQL
   need to act when something changes externally?      → EVENT (webhook)
```

Most real platforms use **all three**: GraphQL to hydrate context, REST to take discrete actions, events to know when to act. Interface selection is per-integration, not per-platform.

## Designing a robust webhook receiver

Event-driven integration is where platforms most often cut corners, so it earns its own treatment. A webhook is an untrusted HTTP call from an external system into yours, and the receiver has four hard requirements:

**1. Verify the source.** Webhooks are public endpoints; anyone can POST to them. Verify the sender's **signature** (an HMAC over the payload with a shared secret) before trusting a byte. An unverified webhook receiver is an open door into your platform.

**2. Return fast; do the work async.** The single most common webhook bug:

```text
   BROKEN                                  CORRECT
   ──────                                  ───────
   receive event                           receive event
   run the 5-minute agent INLINE           verify signature
   ...sender times out at 10s              enqueue {delivery_id, payload}
   sender RETRIES → duplicate run          return 200 immediately
   return 200 (too late)                   worker dequeues → agent runs async
```

Holding the HTTP handshake open while a slow agent runs causes two failures at once: the sender times out and **retries** (so you get duplicate runs), and you have coupled slow work to a fast handshake. **Enqueue and return fast.**

**3. Deduplicate on delivery ID.** Because senders retry on timeout *and* networks duplicate, the same event will arrive more than once. Every webhook carries a unique **delivery ID**; dedupe on it so "delivered twice" equals "processed once." This is the at-least-once idempotency discipline applied at the integration edge.

**4. Tolerate ordering and loss.** Events can arrive out of order or be dropped entirely. Treat each event as a *hint to go check current state* (re-fetch the resource via REST/GraphQL and act on truth) rather than as the authoritative state itself. An event says "something changed" — the pull confirms *what*.

## Worked judgment

*"The agent should review a change and post comments when one is opened."* — Three interfaces, one workflow. **Event:** subscribe to the "change opened" webhook (verify signature, enqueue, return 200, dedupe on delivery ID) — do not poll for new changes. **GraphQL:** hydrate the review context in one round-trip — the diff, the linked issue's status, the existing reviews — rather than an N+1 REST cascade. **REST:** post the review comments (discrete writes). All three, each for the access shape it fits.

*"Just poll the CI API every few seconds to see if the build passed."* — The polling anti-pattern: wasted calls, rate-limit pressure, and you still learn late. Subscribe to the **CI-completion event** instead; the toolchain tells you the moment it finishes, and a verified, fast-returning, deduplicating receiver turns that into an agent action.

## Key takeaways

- Internal platforms integrate with four recurring **toolchain categories** — version control, issue/work tracker, knowledge base, CI/CD — and the category (not the vendor) fixes the integration's shape. Bind behind an MCP server to stay portable and keep credentials out of context.
- **Interface selection per integration:** REST for discrete writes and shallow reads; **GraphQL for deep, related reads in one round-trip** (also a hydration win, but watch the point budget); **events/webhooks to react to external change** instead of polling.
- **Polling to detect change is the anti-pattern**; subscribe to the completion event.
- A robust **webhook receiver** verifies the signature, **returns fast and works async** (inline handling causes sender-retry duplicates), **deduplicates on delivery ID**, and treats each event as a hint to re-check current state rather than authoritative truth.
