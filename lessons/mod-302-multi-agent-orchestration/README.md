# mod-302-multi-agent-orchestration: Multi-Agent Orchestration Architecture

**Estimated effort:** 14 hours

A single agent has one context window, one tool budget, and one line of reasoning. Multi-agent systems break that ceiling — but as an **architect**, your job is not to wire the loops. It is to decide *whether* the system should be multi-agent at all, *which topology* it should take, *who owns the user-facing answer*, *which protocols* carry tools and delegation, and *how the system behaves when a sub-agent stalls, contradicts a peer, or runs away on cost*. This module is about those decisions and the tradeoffs behind them.

> **Architecture, not mechanics.** mod-204 in the AI-Engineer track teaches you to *build* orchestrator-worker loops by hand. This module assumes that skill exists on your team and pitches up: you size the token economics, draw the topology, choose the ownership model, and write the escalation contract that an engineer then implements. The decision artifact — a diagram, a decision matrix, a one-page spec — is the deliverable.

The central tension you will return to throughout the module: a multi-agent system routinely burns **~15x the tokens** of a single chat for the same task. That multiplier buys parallel breadth, isolated context, and specialization. Your designs have to earn it.

## Learning objectives

- **Design orchestrator-worker topologies** and reason quantitatively about their token economics — why fan-out costs roughly 15x a single chat, and when the breadth justifies it.
- **Choose between handoffs and manager-as-tools**, and decide explicitly *who owns the final user-facing answer* in each.
- **Apply inter-agent protocols at the architecture level** — MCP for tool/resource access, A2A for cross-agent delegation — and reason about their boundaries.
- **Define escalation and coordination strategies** — loop bounds, deadlock breaking, human-in-the-loop gates, and partial-failure behavior — across a whole multi-agent system.

## Lecture chapters

1. [Orchestrator-Worker Topologies and Token Economics](01-orchestrator-worker-topologies.md) — fan-out shapes, the 15x multiplier, and how to size a topology to its budget.
2. [Handoffs vs. Manager-as-Tools, and Answer Ownership](02-handoffs-vs-manager-as-tools.md) — two delegation styles, and the architectural question of who speaks to the user.
3. [Inter-Agent Protocols: MCP and A2A](03-inter-agent-protocols.md) — tool access vs. agent delegation, where each protocol belongs, and the seams between them.
4. [Coordination and Escalation Strategy](04-coordination-and-escalation.md) — bounding loops, breaking deadlocks, escalation ladders, and human gates.

## Exercises

Hands-on architecture practice — diagrams, decision matrices, and specs, not framework wiring. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

- [exercise-01: Orchestrator-worker topology design](exercises/exercise-01-orchestrator-worker-topology-design.md) — draw a topology for a real workload and defend its token budget.
- [exercise-02: Handoffs vs. manager-as-tools](exercises/exercise-02-handoffs-vs-manager-as-tools.md) — pick a delegation model and assign answer ownership.
- [exercise-03: MCP/A2A integration architecture](exercises/exercise-03-mcp-a2a-integration-architecture.md) — map tools and delegation onto protocols with explicit contracts.
- [exercise-04: Coordination and escalation design](exercises/exercise-04-coordination-and-escalation-design.md) — write the loop bounds, deadlock rules, and escalation ladder for a system.

## Assessment

- [Quiz 1 — Multi-Agent Orchestration Architecture](quizzes/quiz-01-multi-agent-orchestration.md) — covers all four chapters.

## Prerequisites

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — workflow-vs-agent judgment and Anthropic's orchestration patterns.
- Familiarity with the orchestrator-worker *mechanics* (mod-204 in the AI-Engineer track, or equivalent hands-on experience) so this module can stay at the design altitude.
- Comfort reading token-cost and latency tradeoffs as first-class design inputs.

See [resources.md](resources.md) for primary references.
