# exercise-03: MCP/A2A Integration Architecture

**Estimated effort:** 4 hours

## Objective

Map a multi-agent system's connections onto the right protocols: **MCP** for the tool/resource seams, **A2A** for the agent-delegation seams. You'll classify every edge, write the contracts that cross each seam, and defend the trust boundaries. The deliverable is an integration architecture spec — the document an engineer turns into MCP servers and A2A capability cards.

## Background

This exercise covers material from:

- [Chapter 3 — Inter-Agent Protocols: MCP and A2A](../03-inter-agent-protocols.md)

You decide which seam is a *capability* and which is a *delegation*, then specify the contract. You are not implementing servers.

## Prerequisites

- Chapter 3 read.
- Chapter 1–2 helpful for understanding where delegation edges come from.

## System

> *An incident-response assistant for an SRE team. It must: query the metrics store and the logging system; read and comment on the team's ticketing system; run a read-only database health check; and — when the incident looks security-related — delegate to a separate **security-analysis agent** owned by a different team, which has its own tools and reasoning. A human approves any remediation action.*

## Tasks

### 1. Inventory and classify every edge

- List each connection the assistant needs. For each, classify it **MCP** (a capability you invoke) or **A2A** (an agent you delegate to), and justify with the Chapter 3 test: *could this be a deterministic function?*

### 2. Draw the two-axis architecture

- ASCII diagram with the assistant in the center, MCP edges to tools/resources on one axis and A2A edges to peer agents on the other. Show that the delegated security agent itself reaches *its* tools over MCP (draw both seams).

### 3. Write the contracts

- For each **MCP** edge: tool/resource name, inputs, outputs, errors, side-effects, and the access scope the server runs with.
- For each **A2A** edge: the peer's capability card summary (what it advertises), the task payload, and the task lifecycle you depend on (submitted → working → input-required → completed/failed).

### 4. Trust boundaries and blast radius

- For every edge, state least-privilege scope and what happens if the other side fails, stalls, or returns garbage. The cross-team A2A peer is the highest-risk edge — say how the caller copes.

### 5. Observability across seams

- Specify how a single incident request stays traceable end-to-end: where correlation/trace IDs are injected and propagated across both MCP calls and the A2A task.

## Starter guidance

Use this classification table and the two-axis diagram template.

| Edge | MCP or A2A? | "Could be a function?" | Contract artifact | Trust scope |
| --- | --- | --- | --- | --- |
| metrics store | | | tool schema | read-only |
| logging system | | | tool schema | read-only |
| ticketing read/comment | | | tool schema | scoped write |
| DB health check | | | tool schema | read-only |
| security-analysis agent | | | capability card | cross-team |

```text
                         ┌────────────────────────────┐
   A2A (delegation)      │   incident-response agent    │   MCP (capabilities)
   ┌─────────────────────┤            (you)             ├─────────────────────┐
   ▼                     └────────────────────────────┘                     ▼
┌──────────────────┐                                            ┌────────────────┐
│ security-analysis │  task{incident,scope}                      │ metrics  (RO)  │
│ agent (other team)│  ──lifecycle──▶ result                     │ logs     (RO)  │
│   │ MCP to its    │                                            │ tickets  (RW*) │
│   ▼ own tools     │                                            │ db-health(RO)  │
└──────────────────┘                                            └────────────────┘
                         (human gate before any remediation)
```

The deliverable is the spec (table + diagram + contracts), not running servers.

## Acceptance criteria

You can demonstrate that:

- Every edge is classified MCP or A2A and justified with the could-this-be-a-function test.
- The diagram shows both axes and shows the delegated agent using MCP for its own tools.
- Each MCP edge has a tool/resource contract with inputs, outputs, errors, and access scope; each A2A edge has a capability-card summary and the task lifecycle.
- Least-privilege scope and a failure-coping plan are stated for every edge, with the cross-team A2A peer called out as highest-risk.
- A trace/correlation-ID propagation plan covers both seams end-to-end.

## Reflection

In `NOTES.md`:

1. Which edge was tempting to model as A2A but is really MCP (or vice versa)? What tipped the decision?
2. The security agent is a black box owned by another team. What in your contract survives them rewriting their internals, and what would break?
3. If the A2A peer stalls in `working` forever, what does your assistant do? Tie this to the escalation ladder in Chapter 4.

## Stretch goals

- Version one shared MCP tool and walk through what a breaking schema change does to every agent that calls it. Write the deprecation plan.
- Replace the cross-team A2A agent with an in-house module and argue whether A2A still earns its weight or collapses to an orchestrator-worker call.
- Add a second consuming team that wants to reuse your metrics MCP server and specify the governance (auth, rate limits, audit) the server now needs as a shared choke point.
