# mod-306-guardrails-safety-security: Guardrails, Safety & Security

**Estimated effort:** 20 hours

An agent is a program that takes untrusted text and turns it into privileged actions. That single sentence is the whole security problem. The model reads a web page, a support ticket, a retrieved document — content an attacker may control — and then decides to call a tool that spends money, deletes a row, or emails a customer. Nothing about a clever prompt makes that boundary safe. The boundary is an *architecture*: where you place guardrails, how you scope tool permissions, who has to approve a destructive action, and how untrusted content is kept from ever sitting in the same trust zone as your tools.

This is the largest module in the role because it is the one that gets you paged at 3 a.m. — or sued. You will not be writing a single regex filter and calling it "safety." You will design two-sided moderation, threat-model prompt injection (including the indirect kind that arrives through a retrieved document), draw least-privilege permission matrices, design OAuth and RBAC for agent toolchains, sandbox local and remote CLI agents so a compromised one cannot exfiltrate or destroy, and bake adversarial testing into the system rather than bolting it on after an incident. The anchor texts are the **OWASP Top 10 for LLM Applications 2025** (especially LLM01 prompt injection and LLM06 excessive agency) and Anthropic's safety guidance.

> **Architecture over patches.** Your deliverables here are guardrail-placement diagrams, threat models, permission matrices, token-lifecycle designs, sandbox topologies, and a red-team plan — not a list of banned phrases. A safety property you cannot draw on a whiteboard is a safety property you cannot defend in an incident review.

## Learning objectives

- Design **two-sided input/output moderation** and **per-tool-call guardrails** as explicit, placed components in the request path.
- Mitigate **OWASP LLM01:2025** (prompt injection, including indirect) and **LLM06:2025** (excessive agency) with layered, defense-in-depth controls.
- Apply **least-privilege tool access**, tool sandboxing, and **human approval** for high-risk actions.
- Architect secure **authentication and authorization** for agent toolchains: OAuth flows, token management, and Role-Based Access Control (RBAC).
- Design **sandboxing and strict permission scopes** for local and remote agent/CLI execution to prevent destructive actions or data leakage.
- Architect **segregation of untrusted content** and **adversarial testing** into the system from day one.

## Lecture chapters

1. [Guardrail Placement: Two-Sided Moderation and Per-Tool-Call Checks](01-guardrail-placement.md) — where in the request path each guardrail lives, and why a single filter is not a design.
2. [Prompt Injection: LLM01 and the Indirect Threat](02-prompt-injection-llm01.md) — direct vs. indirect injection, why prevention is impossible and containment is the goal.
3. [Least Privilege, Sandboxing, and Human Approval](03-least-privilege-and-approval.md) — minimal tool grants, capability tokens, and the high-risk-action approval gate.
4. [OAuth, Tokens, and RBAC for Agent Toolchains](04-oauth-rbac-tokens.md) — delegated authorization, token lifecycle, and role-based access control for tools.
5. [Sandboxing Local and Remote CLI Agents](05-sandboxing-cli-agents.md) — process isolation, egress control, and blast-radius limits for code-executing agents.
6. [Untrusted-Content Segregation and Adversarial Testing](06-untrusted-content-and-red-teaming.md) — trust zones, the dual-LLM pattern, and building red-teaming into the lifecycle.

## Exercises

Hands-on design practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

- [exercise-01: Guardrail placement architecture](exercises/exercise-01-guardrail-placement-architecture.md) — place input, output, and per-tool-call guardrails on a real request path and justify each.
- [exercise-02: Prompt injection threat model](exercises/exercise-02-prompt-injection-threat-model.md) — STRIDE-style threat model for a RAG agent exposed to indirect injection.
- [exercise-03: Least-privilege tool permissions](exercises/exercise-03-least-privilege-tool-permissions.md) — build a permission matrix and a tiered approval gate for a tool catalog.
- [exercise-04: OAuth, RBAC, and token management for toolchains](exercises/exercise-04-oauth-rbac-token-management-for-toolchains.md) — design the delegated-auth and token-lifecycle architecture for a multi-tool agent.
- [exercise-05: CLI agent sandboxing, local and remote](exercises/exercise-05-cli-agent-sandboxing-local-and-remote.md) — design isolation and egress control for a code-executing agent in both topologies.
- [exercise-06: Excessive agency controls](exercises/exercise-06-excessive-agency-controls.md) — find and close the LLM06 excessive-agency gaps in a given system.

## Prerequisites

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — the tool/sub-agent/hardcoded-path boundary vocabulary you will be securing.
- Working familiarity with how an agent loop calls tools (the reason-act loop, structured tool calls). You secure the boundary; you must be able to read it.
- Comfort with threat-modeling notation (data-flow diagrams, trust boundaries) and reading OAuth 2.x flow diagrams.
- See [PREREQUISITES.md](../../PREREQUISITES.md) for the role-level entry skills.

See [resources.md](resources.md) for primary references. The OWASP Top 10 for LLM Applications 2025 is the anchor text for this module.
