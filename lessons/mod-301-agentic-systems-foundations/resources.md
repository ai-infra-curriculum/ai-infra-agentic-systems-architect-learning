# Resources for mod-301-agentic-systems-foundations

Primary references for agentic-systems foundations and the workflow-vs-agent decision. Verify against current docs — agent tooling and guidance move fast.

## Anchor text

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the canonical source for this module: the workflow-vs-agent distinction, the five orchestration patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer), and the "start with the simplest solution, add complexity only when it improves outcomes" discipline. Read this first and return to it for every exercise.

## Design judgment and real systems

- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — a documented orchestrator-worker architecture with its token economics and failure modes; the teardown subject for exercise-04.
- **Anthropic — Writing tools for agents / tool design guidance** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) — background for the tool-boundary decisions in Chapter 3 (clear naming, typed schemas, recoverable errors). Browse the engineering blog index for the current tool-design articles.

## Patterns and protocols (context for the boundaries)

- **Model Context Protocol** ([modelcontextprotocol.io](https://modelcontextprotocol.io)) — the spec for connecting agents to shared tools and resources; relevant when a capability becomes a tool that crosses a process boundary. Explored at architecture depth in [mod-302](../mod-302-multi-agent-orchestration/README.md).

## Decision records

- **Architecture Decision Records (ADRs)** ([adr.github.io](https://adr.github.io)) — the format your exercise deliverables use: context → decision → consequences. Several exercises ask you to record contestable boundary calls as ADRs.

## Where this leads

- **[mod-302: Multi-Agent Orchestration Architecture](../mod-302-multi-agent-orchestration/README.md)** — takes the orchestrator-workers pattern from this module to full topology-design depth, including inter-agent protocols and token economics.
- **[mod-304: Evaluation Harnesses](../mod-304-evaluation-harnesses/README.md)** — the evidence side of the "start simple" gate: how you measure whether added complexity actually improved outcomes.

> This module is about judgment, not code. Treat every reference as input to a design decision you must be able to defend in a review.
