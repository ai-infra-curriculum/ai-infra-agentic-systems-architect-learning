# exercise-03: Workflow Toolchain Integration

**Estimated effort:** 3 hours

## Objective

Design the **integration layer** for an internal agent platform that must read from, write to, and react to the org's workflow toolchains — choosing the right interface (REST, GraphQL, or event-driven) **per integration** and designing a robust event receiver. The deliverable is an integration contract: a per-integration interface-selection table with justifications, one shaped GraphQL read, and a webhook-receiver design that survives retries and duplicates. Vendor-neutral by construction — design to the toolchain *category*, then note the specific product.

This is an **architecture** exercise. You will not stand up working webhooks; you will produce the contract and the receiver design.

## Background

This exercise covers material from:

- [Chapter 3 — Workflow & Toolchain Integration](../03-workflow-toolchain-integration.md)

Assume your team can already call a REST API and verify an HMAC signature (the L30 skill). Your job is the layer above: which interface each integration should use, and how the event path is made robust.

## The scenario

Your platform's agent must integrate with the org's toolchains to do a reactive workflow end-to-end. Use this generic shape (substitute the specific products your org runs — the design must not depend on the brand):

> When a unit of work appears (a change opened, or — in a non-SDLC binding — a ticket/alert created), the agent must: (1) **learn about it immediately**, (2) **gather rich related context** about it and its linked items in as few round-trips as possible, (3) **take discrete actions** (comment, transition, update), and (4) **react when a downstream step finishes** (CI completes, or a dependent task closes).

Name the four toolchain **categories** you are integrating: a **version-control or work source**, an **issue/work tracker**, a **knowledge base**, and a **CI/CD or pipeline** system. (A non-SDLC binding may map these to monitoring/ticketing/runbooks/automation — say so.)

## Tasks

### 1. Build the interface-selection table

- For each integration point in the workflow, choose **REST**, **GraphQL**, or **event-driven**, and justify by the *access shape* (discrete write / shallow read → REST; deep related read in one round-trip → GraphQL; react to external change → event), per [Chapter 3](../03-workflow-toolchain-integration.md).

| Integration point | Access shape | Interface | Why this interface (not the others) |
| --- | --- | --- | --- |

- At least one point must be REST (a discrete write), at least one GraphQL (a deep related read), and at least one event-driven (a reaction). State explicitly why the reactive point is **not** solved by polling.

### 2. Shape the GraphQL read

- Take the "gather rich related context" step and write the **single shaped query** that fetches the work item and its related items (reviews/checks/linked issue, or alerts/related tickets) in **one round-trip**.
- Show the **N+1 REST cascade** it replaces, and state how many round-trips you saved.
- Note the GraphQL **point/cost budget** consideration: confirm you are not over-selecting fields you do not need (this is also a context-hydration win — [Chapter 2](../02-secure-tool-use-and-context-hydration.md)).

### 3. Design the robust event receiver

- Specify the webhook/event receiver for the reactive trigger, covering all four hard requirements from [Chapter 3](../03-workflow-toolchain-integration.md):
  - **signature verification** (reject unverified senders),
  - **fast return + async work** (why running the agent inline before returning `200` breaks),
  - **dedupe on delivery ID** (because retries + network duplicates),
  - **tolerate ordering/loss** (treat the event as a hint to re-check current state, not as truth).

| Requirement | Your design | What breaks without it |
| --- | --- | --- |

- Draw the flow: event in → verify → enqueue → return `200` → worker → agent. Mark where a naive inline design would cause **duplicate runs**.

### 4. Tie it back to portability and credentials

- State how each integration is bound behind an **MCP server** ([Chapter 1](../01-extension-model-across-platforms.md)) so the binding stays host-portable and the credential stays out of the model's context ([Chapter 2](../02-secure-tool-use-and-context-hydration.md)).

## Starter guidance

An interface-selection sketch to react to:

```text
   learn about new work immediately   → EVENT (webhook)   ← not polling
   gather item + related context      → GraphQL (1 round-trip)
   comment / transition / update      → REST (discrete writes)
   react when CI / downstream done     → EVENT (webhook)
```

A receiver shape to adapt:

```text
   event in ─▶ verify signature ─▶ enqueue {delivery_id, payload} ─▶ 200
                                                  │
                                    worker ─▶ dedupe(delivery_id) ─▶ agent
   (running the agent INLINE before 200 → sender times out → retries → dup runs)
```

You do **not** need the host-mapping (exercise-01) or the full trust model (exercise-02) here — assume the MCP/credential placement from those and focus on interface selection and the receiver.

## Acceptance criteria

You can demonstrate that:

- The interface-selection table covers every integration point, uses **all three** styles at least once, and justifies each by access shape — with the reactive point explicitly *not* polled.
- The shaped GraphQL query fetches the item and its related items in **one round-trip**, the replaced N+1 cascade is shown with a round-trip count, and over-selection is avoided.
- The event receiver design satisfies all four requirements, and the flow diagram marks where a naive inline design causes duplicate runs.
- Each integration is bound behind an MCP server with the credential out of context.

## Reflection

In `NOTES.md`:

1. Which integration point was genuinely ambiguous between two interfaces, and what tipped it?
2. Trace what happens end-to-end if your event sender retries three times on a slow handshake *without* your dedupe — how many agent runs fire, and how does dedupe-on-delivery-ID fix it?
3. Where would polling have been "good enough," and what specifically convinced you the event-driven design was worth the extra receiver complexity?

## Stretch goals

- Add a **rate-budget analysis**: estimate the GraphQL point cost of your shaped query versus the REST call count of the cascade, and say which the toolchain's limits actually favor.
- Design the **idempotency** of a discrete REST write (e.g., "comment on the item") so that a re-driven event does not double-post — reusing the at-least-once discipline.
- Re-bind the whole integration to a **non-SDLC** toolchain (monitoring + paging + ticketing + automation) and confirm the interface-selection rule is unchanged ([Chapter 5](../05-beyond-sdlc-generalizing-the-platform.md)).
