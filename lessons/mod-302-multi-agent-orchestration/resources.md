# Resources for mod-302-multi-agent-orchestration

Primary references for multi-agent orchestration architecture. Read them as an architect — for the *decisions and tradeoffs*, not the API surface. Verify against current docs; agent tooling and protocols move fast.

## Patterns, topologies, and economics

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the canonical taxonomy: prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer, and the "start simple, add complexity only when it pays" discipline. Start here.
- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — a real orchestrator-worker system, including the **token economics** (agents ≈ 4x a chat; multi-agent ≈ 15x), the context-economics of distilled returns, and the coordination/failure modes. The source for this module's 15x framing.

## Protocols

- **Model Context Protocol (MCP)** ([modelcontextprotocol.io](https://modelcontextprotocol.io)) — the open standard for connecting agents to tools, data sources, and resources. Read the architecture and core concepts for the *capability* seam; the contract (tool schema, resources) is the load-bearing part.
- **Agent2Agent (A2A) Protocol** ([a2a-protocol.org](https://a2a-protocol.org)) — an open protocol for agent-to-agent delegation: capability cards, task lifecycle, and the framework-agnostic delegation contract for the *agent-delegation* seam. Treat the pattern (typed tasks, capability advertisement, lifecycle) as the lesson, independent of any one implementation.

## Frameworks (for grounding the patterns)

- **OpenAI Agents SDK** — handoffs and routing as first-class primitives; a concrete reference for the Chapter 2 control-transfer model.
- **LangGraph** — graph-structured multi-agent orchestration with explicit, inspectable state; useful for reasoning about coordination and bounded loops.
- **CrewAI / AutoGen** — role-based multi-agent crews; reference points for the manager-as-tools style and role specialization.
- **Claude Agent SDK** — sub-agent isolation and tool scoping built in; concrete grounding for distilled returns and least-privilege tool surfaces.

> You are designing topologies, protocol seams, and escalation contracts here — the frameworks above *package* these patterns. Recognize which pattern each one is implementing, and your spec will translate cleanly into whichever framework a team adopts. See [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) for the workflow-vs-agent judgment that precedes any of this.
