# mod-310-agentic-developer-platforms: Agentic Developer Platforms & SDLC Integration

**Estimated effort:** 16 hours

Most teams adopt an agentic coding tool the way they adopt any IDE: install it, point it at the repo, and let each engineer configure it ad hoc. That works for one developer. It does not work for a 200-engineer org with a private monorepo, an internal CI system, a Jira instance full of context, and a security team that wants to know exactly which systems an agent can touch. Turning a personal coding assistant into an **organizational developer platform** is an architecture problem — and that is what this module is about.

You will design a **plugin-based platform** on top of an agentic coding tool (we use Claude Code as the concrete reference, because its extension standards — Agents, Skills, Hooks, and the Model Context Protocol — are public and well-documented at [code.claude.com/docs](https://code.claude.com/docs)). Then you will architect the harder parts: securely interfacing the agent with internal SDLC toolchains and proprietary data, building reliable tool-use and context-hydration patterns, integrating with GitHub/Jira/Confluence/CI over the right interface for each, and — the part most platform teams under-invest in — designing the developer experience that makes the thing get adopted instead of bypassed.

> **Architecture, not mechanics.** The AI-Engineer track teaches you to *build* an MCP server or write a hook. This module pitches up: you decide *which* extension point a capability belongs in, *where* the trust boundary sits, *which* protocol carries each integration, and *what* the onboarding path is. The deliverables are architecture diagrams, integration contracts, threat models, and adoption plans — backed by enough real config and code that an engineer could implement them without guessing.

The platform you are designing is the one a real posting describes: an agent that does requirements analysis, code generation, and debugging against an org's *own* toolchain and *own* data — safely, reliably, and in a way developers actually want to use.

## Learning objectives

- **Design a plugin-based architecture** that extends an agentic coding tool using its four extension standards — Agents (subagents), Skills, Hooks, and MCP — choosing the right extension point for each capability and packaging them as a distributable plugin.
- **Architect AI for the SDLC**: securely interface an agent with internal SDLC toolchains and proprietary data (source, tickets, design docs) to perform requirements analysis, code generation, and debugging, with explicit trust boundaries and least-privilege access.
- **Build reliable tool-use patterns and context-hydration strategies** that give an agent precise, just-enough system access and context for SDLC tasks — and degrade safely when context is missing.
- **Integrate with core DevOps and project-management toolchains** (GitHub, Jira, Confluence, CI/CD) over REST, GraphQL, and event-driven interfaces, choosing the right interface per integration.
- **Design developer-focused tooling** with first-class usability: adoption, onboarding, documentation, and continuous feedback loops that make the platform spread instead of stall.

## Lecture chapters

1. [Plugin Architecture on Agents, Skills, Hooks, and MCP](01-plugin-architecture-extension-standards.md) — the four extension points, what belongs in each, and how a plugin packages them for an org.
2. [Architecting AI for the SDLC: Trust Boundaries and Proprietary Data](02-ai-for-sdlc-trust-boundaries.md) — interfacing an agent with internal toolchains and private data for requirements, code-gen, and debugging — safely.
3. [Reliable Tool-Use Patterns and Context Hydration](03-tool-use-and-context-hydration.md) — designing tools the model uses correctly, and hydrating just-enough context without blowing the window.
4. [Integrating DevOps and PM Toolchains: REST, GraphQL, Event-Driven](04-devops-pm-toolchain-integration.md) — GitHub, Jira, Confluence, and CI/CD, and choosing the interface per integration.
5. [Developer Experience: Adoption, Onboarding, and Feedback Loops](05-developer-experience-and-adoption.md) — why platforms get bypassed, and the DX architecture that prevents it.

## Exercises

Hands-on architecture practice — diagrams, integration contracts, threat models, and real config, not framework wiring. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

- [exercise-01: Plugin architecture on Agents/Skills/Hooks/MCP](exercises/exercise-01-plugin-architecture-agents-skills-hooks-mcp.md) — design and partially build a plugin that maps capabilities onto the four extension points.
- [exercise-02: AI-for-SDLC tool-use patterns](exercises/exercise-02-ai-for-sdlc-tool-use-patterns.md) — architect a secure agent interface to an internal SDLC toolchain with explicit trust boundaries.
- [exercise-03: Context-hydration strategies](exercises/exercise-03-context-hydration-strategies.md) — design a layered hydration strategy for an SDLC task and defend its context budget.
- [exercise-04: DevOps toolchain integration](exercises/exercise-04-devops-toolchain-integration.md) — integrate GitHub, Jira, and CI over REST/GraphQL/events with per-interface justification.
- [exercise-05: Developer tooling adoption and DX](exercises/exercise-05-developer-tooling-adoption-and-dx.md) — design the onboarding, documentation, and feedback architecture for the platform.

## Assessment

- [Quiz 1 — Agentic Developer Platforms & SDLC Integration](quizzes/quiz-01-agentic-developer-platforms.md) — covers all five chapters.

## Prerequisites

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — workflow-vs-agent judgment and orchestration patterns.
- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — subagents, MCP, and A2A at the architecture level.
- Familiarity with the *mechanics* of MCP servers, hooks, and tool definitions (mod-204 in the AI-Engineer track, or equivalent hands-on experience) so this module can stay at the design altitude.
- Working knowledge of at least one DevOps/PM toolchain (GitHub, Jira, or a CI system) as an API consumer.

See [resources.md](resources.md) for primary references.
</content>
</invoke>
