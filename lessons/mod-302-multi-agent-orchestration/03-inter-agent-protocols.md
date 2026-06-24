# Chapter 3 — Inter-Agent Protocols: MCP and A2A

A multi-agent system has two fundamentally different kinds of connection, and conflating them is one of the most common architecture mistakes. One connects an agent to **tools and resources** it uses. The other connects an agent to **another agent** it delegates to. **MCP** is the standard for the first; **A2A** is an emerging open standard for the second. As an architect you decide which seam is which and what contract crosses it.

## Two axes, two protocols

```text
        ┌──────────────────────────────────────────────┐
        │                 AGENT                         │
        │                                               │
        │   ── A2A ──▶  another agent (delegation)       │
        │       "do this task, report back"             │
        │                                               │
        │   ── MCP ──▶  tool / resource (capability)     │
        │       "call this function, read this data"     │
        └──────────────────────────────────────────────┘
```

- **MCP (Model Context Protocol)** standardizes how an agent connects to **tools, data sources, and resources**. Think "USB-C for tools": one client-server contract so any agent can use any MCP-exposed capability — a database, a search API, a file store — without bespoke glue. The unit of exchange is a *tool call / resource read*.
- **A2A (Agent2Agent)** standardizes how one agent **delegates a task to another agent** and tracks it. The other agent is opaque — you don't see its internal tools or reasoning, only its advertised capabilities (an "agent card") and the task lifecycle. The unit of exchange is a *task with a lifecycle* (submitted → working → input-required → completed/failed), not a single function return.

The key distinction: **MCP gives an agent capabilities; A2A lets an agent hand work to a peer that has its own capabilities.** A tool does what you tell it. An agent decides how to accomplish what you asked.

## Where each belongs

| Concern | MCP (tools/resources) | A2A (agent delegation) |
| --- | --- | --- |
| Other side is | A function/resource you invoke | An autonomous agent you task |
| Exchange unit | Tool call → result | Task → lifecycle → result |
| Opacity | You see the schema and call it | Peer is a black box behind a capability card |
| State | Mostly stateless per call | Stateful, long-running task with status |
| Use when | Agent needs a capability (search, DB, calc) | Agent needs *another agent's* judgment/work |
| Trust boundary | You own/vet the tool | Peer may be a different team/vendor/org |

A useful test: *if you'd be comfortable replacing the other side with a deterministic function, it's a tool — use MCP. If the other side needs to reason, plan, or use its own tools to satisfy the request, it's an agent — use A2A.*

## Architecture seams these protocols clarify

- **Tool sharing without duplication.** Expose a capability once over MCP; every agent that needs it connects to the same server instead of each re-implementing it. The MCP server becomes a governance choke point — auth, rate limits, and audit live there.
- **Cross-team / cross-vendor delegation.** When the agent you delegate to is owned by another team or vendor, A2A's capability card and task lifecycle give you a contract that survives their internal changes. You depend on the *card*, not their implementation.
- **Composition over a bus.** A2A lets you assemble agents built in different frameworks and even different organizations into one system, because the delegation contract is framework-agnostic.

The two often coexist in one system: an A2A-delegated agent internally uses MCP tools to do its job. You delegate *to it* over A2A; *it* reaches its database over MCP. Draw both seams.

## Architect-level concerns

- **The interface is the hard part.** Whether MCP tool schema or A2A capability card, the contract is where systems break. Specify inputs, outputs, errors, and side-effects as tightly as a public API — version it, and treat a breaking change to a shared MCP tool as a breaking change to every agent that calls it.
- **Trust and blast radius.** An MCP tool runs with the access you grant the server; an A2A peer runs autonomously on your behalf. Scope each to least privilege, and assume an A2A peer can fail, stall, or return garbage — design the calling agent to cope (Chapter 4).
- **Don't reach for A2A when MCP suffices.** Wrapping a deterministic capability as a full agent buys you an autonomous black box and a heavier lifecycle for nothing. Most "agent-to-agent" needs inside one system are really orchestrator-to-worker calls; A2A earns its weight at *organizational* boundaries.
- **Observability across the seam.** Propagate trace/correlation IDs across both MCP calls and A2A tasks so a single user request is traceable end to end — otherwise a failure inside a delegated agent is invisible from the caller.

## Key takeaways

- Two distinct connections: **MCP** connects an agent to **tools/resources** (a capability you invoke); **A2A** connects an agent to **another agent** (a task you delegate).
- Test: if it could be a deterministic function, it's MCP; if it must reason/plan with its own tools, it's A2A.
- The **contract** — tool schema or capability card — is the load-bearing part; version it and treat changes as breaking.
- A2A earns its weight at organizational/vendor boundaries; inside one system, most "agent-to-agent" work is just orchestrator-to-worker and doesn't need it. Propagate trace IDs across both seams.
