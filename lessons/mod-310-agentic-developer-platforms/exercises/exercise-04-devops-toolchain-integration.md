# exercise-04: DevOps and PM Toolchain Integration

**Estimated effort:** 3 hours

## Objective

Architect the integration layer of the platform: connect GitHub, Jira, Confluence, and a CI system, choosing the right interface (REST, GraphQL, or event-driven) for each integration and justifying every choice by the shape of the data and the trigger. You will produce an integration spec table and an event-driven backbone diagram for a real SDLC loop, including the robustness controls (queueing, event verification, idempotency, rate-limit handling). The deliverable is an integration architecture an engineer could wire up.

## Background

This exercise covers material from:

- [Chapter 4 — Integrating DevOps and PM Toolchains: REST, GraphQL, Event-Driven](../04-devops-pm-toolchain-integration.md)
- [Chapter 2 — Trust Boundaries](../02-ai-for-sdlc-trust-boundaries.md) (the agent reaches these through MCP tools with boundary controls)

The selection rule: **REST for writes and shallow reads; GraphQL for deep/related reads in one round-trip; events for reacting to change without polling.**

## Prerequisites

- Read Chapter 4; skim the GitHub REST and GraphQL API docs and the Jira/Confluence REST API docs (links in `resources.md`).
- Pick the toolchain: GitHub + Jira + Confluence + one CI system (GitHub Actions, Jenkins, or similar).

## The loop to architect

*"When a Jira ticket is assigned to the bot, implement it and open a PR. When CI fails on that PR, kick off a debug pass. When it's green, comment back on the ticket."*

## Tasks

### 1. Choose an interface per integration

- For each integration (read ticket, read design doc, gather code context, open PR, comment on PR, transition ticket, trigger/read CI), pick REST, GraphQL, or event-driven, and justify it in one line against the selection rule.
- Find at least one place where GraphQL collapses an N+1 REST cascade into one query, and at least one place where an event replaces polling.

### 2. Draw the event-driven backbone

- Diagram the loop: the Jira "assigned" webhook → enqueue → pull context → open PR → CI "completed" webhook → debug/comment. Show which arrows are REST, which are GraphQL, which are events.

### 3. Design webhook robustness

- Specify: the handler enqueues and returns `200` fast (doesn't run the agent inline); webhook signatures are verified; events are deduplicated on delivery ID (at-least-once delivery means duplicates will happen).

### 4. Treat rate limits as a design input

- Identify where the platform is chattiest, and specify the mitigations: GraphQL batching, caching stable reads, backoff on `429`. State the rate budget as a constraint, not an afterthought.

### 5. Connect to the trust boundary

- Confirm each integration is reached through an MCP tool (not raw), and that webhook payloads are treated as untrusted input (Chapter 2).

### 6. Write the spec

- Produce the table: `trigger/action | interface | calls | why this interface`.

## Starter guidance

Target spec shape:

```text
| Trigger / action          | Interface       | Calls                          | Why                          |
| ------------------------- | --------------- | ------------------------------ | ---------------------------- |
| Ticket assigned to bot    | Jira webhook    | enqueue task                   | react to change, don't poll  |
| Read ticket + design      | Jira/Conf REST  | get_issue, get_page            | shallow reads                |
| Gather code context       | GitHub GraphQL  | repo + files + linked issue,   | deep related read, no N+1    |
|                           |                 | one query                      |                              |
| Open PR                   | GitHub REST     | POST /pulls                    | a write                      |
| CI finished / failed      | CI webhook      | enqueue debug task             | react to completion          |
| Read failing logs         | CI REST         | get_job_log                    | shallow read                 |
| Comment back on ticket    | Jira REST       | add_comment                    | a write                      |
```

Backbone sketch:

```text
Jira webhook ──▶ [verify sig] ──▶ [dedup on delivery id] ──▶ [enqueue, return 200]
                                                                   │
                                                                   ▼
                                                          run SDLC agent
                                                       (REST/GraphQL via MCP)
                                                                   │
                                          GitHub "check completed" webhook ◀─┘
```

## Acceptance criteria

You can demonstrate that:

- Every integration has an interface choice justified in one line by the selection rule.
- At least one GraphQL query replaces an N+1 REST cascade, and at least one event replaces a polling loop, each called out explicitly.
- The webhook backbone enqueues-and-returns-fast, verifies signatures, and deduplicates on delivery ID.
- Rate limits are named as a constraint with concrete mitigations (batching, caching, backoff).
- Each integration is reached through an MCP tool, and webhook payloads are treated as untrusted.

## Reflection

In `NOTES.md`:

1. Which integration was the closest call between REST and GraphQL, and what tipped it?
2. What breaks if the webhook handler runs the agent *inline* instead of enqueuing? Walk through a slow run and a retry.
3. Where would you hit a rate limit first under load, and what's your first mitigation?

## Stretch goals

- Add a "ticket reassigned away from bot mid-run" race and specify how the platform cancels or ignores the stale task.
- Specify the dead-letter handling for events that fail processing repeatedly — where do they go, and who looks at them?
- Add GitHub's GraphQL point-budget math for your context-gathering query and show it stays within a sane per-run budget.
</content>
