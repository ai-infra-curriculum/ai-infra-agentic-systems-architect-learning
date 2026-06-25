# mod-310-agentic-developer-platforms: Agentic Developer & Internal Platforms

**Estimated effort:** 16 hours

🎯 **Purpose.** Most teams adopt an agentic tool the way they adopt any editor: install it, point it at a repo or a queue, and let each person configure it ad hoc. That works for one user. It does not work for a 200-person org with private systems, internal services, a ticketing system full of context, and a security team that wants to know exactly which systems an agent can touch. Turning a personal assistant into an **organizational platform** is an architecture problem — and it is the same problem whether the agent writes code, triages incidents, or answers support tickets. This module teaches you to design that platform *generically*, across multiple host platforms as peers, and to prove the pattern is not specific to software development.

> **Architecture, not mechanics.** The L30 AI-Engineer track teaches you to *build* an MCP server, write a hook, or wire a subagent on one specific tool. This module pitches up: you decide *which* extension point a capability belongs in, *where* the trust boundary sits, *which* protocol carries each integration, *how portable* the design is across hosts, and *what* the onboarding path is. The deliverables are architecture diagrams, integration contracts, threat models, portability/lock-in grades, and adoption plans — backed by enough real config that an engineer could implement them on **any** host without guessing.

The central claim of the module: **an agentic platform is a host-independent architecture with a thin, host-specific binding at the edges.** The capabilities (agents, tools/skills, lifecycle hooks, external access via MCP) recur across Claude Code, Cursor, GitHub Copilot, LangGraph Platform, and a custom in-house host. Where they live and how portable they are differs — and grading that difference is a core architect skill. Software development (SDLC) is *one worked example* of the platform; the final chapter applies the identical architecture to a non-developer internal domain (ops/support/analytics) to prove it generalizes.

## Learning objectives

- **Design an extensible agent-platform architecture** using a generic extension model — agents, skills/tools, hooks/lifecycle events, and MCP — and map those concepts across **at least two real platforms as peers** (e.g., Claude Code, Cursor, Copilot, LangGraph Platform, a custom host), grading portability and lock-in for each.
- **Architect agents that interface securely with internal toolchains and proprietary data** to perform real work — analysis, generation, debugging, operations — with explicit trust boundaries and least-privilege access. SDLC is one worked example, not the only context.
- **Build reliable tool-use and context-hydration patterns** that give an agent precise, scoped system access and just-enough context, and degrade safely when context is missing.
- **Integrate with core workflow toolchains** — version control, issue trackers, knowledge bases, CI/CD — over REST, GraphQL, and event-driven interfaces, choosing the right interface per integration, vendor-neutrally.
- **Design platform-level developer experience** — onboarding, adoption, documentation, versioning, and feedback loops — that makes the platform spread instead of stall, and that generalizes to non-developer users.

## Lecture chapters

1. [The Extension Model Across Platforms](01-extension-model-across-platforms.md) — the four generic extension points (agents, skills/tools, hooks/lifecycle, MCP), mapped across named platforms as peers, with portability and lock-in seams graded.
2. [Secure Tool-Use and Context Hydration](02-secure-tool-use-and-context-hydration.md) — reliable, scoped tool-use and just-enough context for an agent acting on internal data, with trust boundaries and safe degradation.
3. [Workflow & Toolchain Integration](03-workflow-toolchain-integration.md) — integrating VCS, issue trackers, knowledge bases, and CI/CD over REST, GraphQL, and event-driven interfaces, choosing the right interface per integration.
4. [Platform Developer Experience and Adoption](04-platform-developer-experience-and-adoption.md) — onboarding, docs, versioning, and feedback loops for an internal agent platform, and why platforms get bypassed.
5. [Beyond SDLC: Generalizing the Platform](05-beyond-sdlc-generalizing-the-platform.md) — applying the same platform pattern to a non-developer internal domain (ops/support/analytics) to prove the architecture generalizes.

## Exercises

Hands-on architecture practice — mapping tables, integration contracts, threat models, portability grades, and real config, not framework wiring. Reference solutions live in the paired [solutions repo](https://github.com/ai-engineering-curriculum/agentic-systems-architect-solutions).

| Exercise | Focus | Effort |
| --- | --- | --- |
| [exercise-01: Extension-model mapping across platforms](exercises/exercise-01-extension-model-mapping-across-platforms.md) | Map the same extension architecture onto two+ named platforms as peers; grade portability and lock-in | 4h |
| [exercise-02: Secure tool-use and context hydration](exercises/exercise-02-secure-tool-use-and-context-hydration.md) | Scoped access + layered context hydration for an agent on internal data | 4h |
| [exercise-03: Workflow toolchain integration](exercises/exercise-03-workflow-toolchain-integration.md) | Integrate VCS / issue-tracker / CI-CD over REST + GraphQL + events, vendor-neutral | 3h |
| [exercise-04: Platform adoption and DX plan](exercises/exercise-04-platform-adoption-and-dx-plan.md) | Onboarding, docs, versioning, and feedback loops for an internal agent platform | 3h |
| [exercise-05: Non-SDLC platform case](exercises/exercise-05-non-sdlc-platform-case.md) | Apply the platform pattern to a non-developer internal domain; prove generalization | 2h |

## Assessment

- [Quiz 1 — Agentic Developer & Internal Platforms](quizzes/quiz-01-agentic-developer-platforms.md) — covers all five chapters; no question assumes one platform or one context.

## Prerequisites

This module sits at L48 in the **Agentic Systems Architect** track and builds directly on the **L30 [agentic-ai-engineer](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) track**, which teaches the *mechanics* this module designs above:

- **L30 agentic-ai-engineer** — building an MCP server, writing a hook, defining a tool, wiring a subagent on a concrete host. This module assumes you can do all of that and stays at the design altitude.
- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — workflow-vs-agent judgment and orchestration patterns.
- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — subagents, MCP, and agent-to-agent interfaces at the architecture level.
- Working knowledge of at least one workflow toolchain (a VCS host, an issue tracker, or a CI system) as an API consumer.

See [resources.md](resources.md) for primary references across platforms.
