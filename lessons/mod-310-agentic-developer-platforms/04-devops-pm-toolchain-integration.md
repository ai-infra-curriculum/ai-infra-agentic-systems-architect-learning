# Chapter 4 — Integrating DevOps and PM Toolchains: REST, GraphQL, Event-Driven

The platform's value is realized at its integrations. An agent that can read a Jira ticket, find the relevant code, open a GitHub PR, watch CI, and update the ticket has closed a real SDLC loop — but only if each of those integrations is built over the *right* interface. GitHub, Jira, Confluence, and a CI system each expose multiple interfaces, and choosing wrong (polling REST when you should subscribe to an event; firing N REST calls when one GraphQL query would do) produces an integration that is slow, rate-limited, or brittle. This chapter is about choosing the interface per integration.

## Three interface shapes

```text
  REQUEST/RESPONSE                          PUSH
  ────────────────                          ────
  ┌──────────┐   ┌──────────┐        ┌──────────┐
  │   REST   │   │ GraphQL  │        │  EVENTS  │
  │ resource-│   │ one query│        │ webhook /│
  │ oriented │   │ shaped to│        │ queue /  │
  │ endpoints│   │ your need│        │ stream   │
  └──────────┘   └──────────┘        └──────────┘
   you pull,      you pull,           they push,
   fixed shape    custom shape        you react
        │              │                   │
   simple writes   avoid N+1,          react to state
   & reads,        deep graphs,        changes without
   broad coverage  fewer round-trips   polling
```

- **REST** — resource-oriented endpoints (`GET /issue/{id}`, `POST /pulls`). Ubiquitous, simple, well-understood, broad coverage. The right default for **writes** and for **simple, shallow reads**. Its weakness is over- and under-fetching: you get the endpoint's fixed shape, so a screen of related data can mean a cascade of calls (the N+1 problem).
- **GraphQL** — one endpoint, you send a query shaped to *exactly* the fields and relationships you want. The right choice when you need **deep or related data in one round-trip** — "this PR, its checks, its reviews, the linked issue, and that issue's status" is one GraphQL query versus a fistful of REST calls. GitHub exposes a mature GraphQL API for precisely this; use it to fetch composite read context efficiently.
- **Event-driven** — the toolchain **pushes** to you when something changes (a webhook on PR opened, a CI build finished, an issue transitioned). The right choice when the trigger is *"react to a state change"* rather than *"go ask if anything changed."* Polling REST in a loop to detect "did CI finish yet?" is the anti-pattern; subscribe to the completion event instead.

The selection rule: **REST for writes and shallow reads; GraphQL for deep/related reads in one trip; events for reacting to change without polling.** Most real platforms use all three.

## Mapping the toolchain

How the common tools shake out:

| Tool | Best for | Interface |
| --- | --- | --- |
| **GitHub** | open/update PR, comment, push | REST (writes) |
| | PR + checks + reviews + linked issue in one read | GraphQL |
| | "PR opened", "review submitted", "check completed" triggers | Webhooks (events) |
| **Jira** | read issue, transition status, add comment | REST |
| | "issue assigned to bot", "status → In Progress" triggers | Webhooks |
| **Confluence** | read a design/spec page for requirements context | REST |
| **CI/CD** | trigger a pipeline, read a job's logs | REST |
| | "build finished / failed" → kick off the debug agent | Events (webhook / queue) |

Read this as: **the integration architecture is a set of per-tool interface choices**, justified by the shape of the data and the trigger. The agent never talks to these raw — it talks through MCP tools (Chapter 1) whose servers wrap these interfaces with the trust-boundary controls of Chapter 2.

## The event-driven backbone

The most architecturally interesting integrations are event-driven, because they turn the platform from "an agent an engineer invokes" into "a system that reacts to the SDLC." A canonical loop:

```text
  Jira: ticket assigned to @agent-bot
        │ (webhook)
        ▼
  ┌──────────────┐    pull ticket (REST)     ┌─────────┐
  │  Platform    │──────────────────────────▶│  Jira   │
  │  event       │    pull code (GraphQL)     ├─────────┤
  │  handler     │──────────────────────────▶│ GitHub  │
  │  (queues a   │    open PR (REST)          │         │
  │   task)      │◀──────────────────────────│         │
  └──────┬───────┘                            └─────────┘
         │ runs the SDLC agent (Chapter 2 boundaries)
         ▼
  GitHub: "check completed" (webhook) ──▶ agent reads result, updates Jira
```

Three architectural properties make this backbone robust:

- **Queue, don't block.** The webhook handler should *enqueue* a task and return `200` fast, not run a 5-minute agent inline. Webhook senders retry on timeout and have delivery limits; absorbing the work into a queue decouples the slow agent from the fast HTTP handshake.
- **Verify and idempotency-key every event.** Verify webhook signatures (the event is an untrusted input from Chapter 2). Deduplicate on the event/delivery ID, because at-least-once delivery means you *will* receive duplicates — and you do not want the agent opening the same PR twice.
- **Respect rate limits as a design input.** REST and GraphQL both meter you (GitHub's GraphQL uses a point-based budget; REST uses request counts). Batch with GraphQL where you can, cache stable reads, back off on `429`, and treat the rate budget as a first-class constraint on how chatty the integration is — not something you discover in production.

## A worked integration spec

*"When a ticket is assigned to the bot, implement it and open a PR; when CI fails on that PR, debug it."* The integration spec:

| Trigger | Interface | Calls | Why this interface |
| --- | --- | --- | --- |
| Ticket → bot | Jira **webhook** | enqueue task | react to change, don't poll Jira |
| Read ticket + linked design | Jira/Confluence **REST** | `get_issue`, `get_page` | simple shallow reads |
| Gather code context | GitHub **GraphQL** | repo + relevant files + linked issue, one query | deep related read, avoid N+1 |
| Open PR | GitHub **REST** | `POST /pulls` | a write |
| CI finished/failed | CI **webhook** | enqueue debug task | react to completion, don't poll |
| Read failing logs | CI **REST** | `get_job_log` | a shallow read |

Each row pairs a trigger with the interface its data shape demands. That table is the integration architecture — and producing one, with per-row justification, for a real toolchain is [exercise-04](exercises/exercise-04-devops-toolchain-integration.md).

## Key takeaways

- Pick the interface per integration: **REST for writes and shallow reads, GraphQL for deep/related reads in one round-trip, events for reacting to change without polling** — most platforms use all three.
- Polling REST to detect state changes is the anti-pattern; subscribe to **webhooks/events** instead, and use GraphQL to collapse N+1 REST cascades into one query.
- An **event-driven backbone** turns the platform from invoke-on-demand into a system that reacts to the SDLC — but only if you **queue (don't block), verify-and-deduplicate every event, and treat rate limits as a design input.**
- The agent reaches none of these raw; it goes through **MCP tools** that wrap each interface with the trust-boundary controls from Chapter 2. The integration spec is a per-trigger table of interface choices with justification.
</content>
