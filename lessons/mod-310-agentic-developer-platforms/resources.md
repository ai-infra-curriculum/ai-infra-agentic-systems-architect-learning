# Resources for mod-310-agentic-developer-platforms (Agentic Developer & Internal Platforms)

Primary references for designing an extensible internal agent platform across host platforms as peers. Verify against current docs — agent tooling and platform APIs move fast. No reference below is "the" platform; treat the hosts as equals and design at the generic-extension-model level ([Chapter 1](01-extension-model-across-platforms.md)).

## Host platforms (the extension model, mapped as peers)

These are the named hosts the module maps as peers. Read them side by side to see the *same* four extension points (agents, skills/tools, hooks/lifecycle, MCP) bound differently.

- **Claude Code — docs** ([code.claude.com/docs](https://code.claude.com/docs)) — subagents, Agent Skills, hooks (`PreToolUse`/`PostToolUse`/`Stop`/`SessionStart`), MCP client, and plugins/marketplaces. One peer's full extension model, well-documented.
- **Cursor — docs** ([docs.cursor.com](https://docs.cursor.com)) — rules/`.cursor/rules`, custom modes/agents, built-in tools, and MCP support. A second peer with a different packaging of the same points.
- **GitHub Copilot — docs** ([docs.github.com/copilot](https://docs.github.com/en/copilot)) — coding agent, custom/repository instructions, extensions, and MCP. A third peer; note where the lifecycle/hook row is thin and guarantees move to CI/policy.
- **LangGraph Platform — docs** ([langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/)) — graph nodes/sub-graphs as agents, tools as functions, interrupts/callbacks as the lifecycle layer, and MCP client support. An orchestration-platform peer that makes the lifecycle row explicit.
- **Building a custom host** — the always-available peer. The **Claude Agent SDK** ([code.claude.com/docs/en/sdk](https://code.claude.com/docs/en/sdk)) and equivalent agent frameworks let you implement the four extension points yourself; use this as the portability/lock-in baseline.

## Model Context Protocol (the portability anchor)

- **MCP specification and docs** ([modelcontextprotocol.io](https://modelcontextprotocol.io)) — the open protocol for tools, resources, and prompts. The authoritative reference for the external-access layer and the most portable part of any platform ([Chapter 1](01-extension-model-across-platforms.md)).
- **MCP servers reference** ([github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)) — reference and community server implementations to study before building your own.

## Agent design, tool-use, and context (Chapter 2)

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — workflow-vs-agent judgment and the patterns underpinning a platform's internals; vendor framing but broadly applicable.
- **Anthropic — Equipping agents for the real world with Agent Skills** ([anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)) — skills as loadable procedural knowledge; the "docs that pay twice" idea ([Chapter 4](04-platform-developer-experience-and-adoption.md)).
- **OWASP Top 10 for LLM Applications** ([owasp.org/www-project-top-10-for-large-language-model-applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)) — prompt injection, insecure tool use, excessive agency; the threat catalog behind the trust-asymmetry design ([Chapter 2](02-secure-tool-use-and-context-hydration.md)).

## Toolchain integration: REST, GraphQL, event-driven (Chapter 3)

Vendor-neutral by category. The interface-selection rule (REST writes / GraphQL deep reads / events to react) holds whichever products an org runs; these are representative APIs to study.

- **REST API design — Microsoft API guidelines** ([github.com/microsoft/api-guidelines](https://github.com/microsoft/api-guidelines)) — resource modeling and the shallow-read/discrete-write shape REST fits.
- **GraphQL — official spec & learn** ([graphql.org/learn](https://graphql.org/learn/)) — client-shaped queries that collapse an N+1 read into one round-trip; note query-cost/point budgets.
- **GitHub REST + GraphQL** ([docs.github.com/rest](https://docs.github.com/en/rest), [docs.github.com/graphql](https://docs.github.com/en/graphql)) — one VCS host showing both interfaces side by side; the GraphQL point budget is a concrete example.
- **Atlassian Jira Cloud REST + webhooks** ([developer.atlassian.com/cloud/jira/platform/rest/v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/), [/webhooks](https://developer.atlassian.com/cloud/jira/platform/webhooks/)) — an issue-tracker peer with reads/transitions over REST and events over webhooks.
- **GitLab webhooks** ([docs.gitlab.com/ee/user/project/integrations/webhooks.html](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html)) — a second event-source peer; compare signature/secret handling across vendors.
- **Webhook robustness — verification, idempotency, async handling** ([webhooks.fyi](https://webhooks.fyi/)) — signature verification, fast-return + async processing, and dedupe-on-delivery-ID — the four receiver requirements in [Chapter 3](03-workflow-toolchain-integration.md).

## Developer experience, adoption, and feedback loops (Chapter 4)

- **Team Topologies — platform-as-a-product / thinnest viable platform** ([teamtopologies.com](https://teamtopologies.com/)) — treating an internal platform as a product with adoption and DX as first-class concerns; the framing behind "platforms get bypassed."
- **Internal Developer Platform — concepts** ([internaldeveloperplatform.org](https://internaldeveloperplatform.org/)) — golden paths, self-service, and onboarding patterns that generalize to agent platforms.
- **Anthropic — Claude Code best practices** ([anthropic.com/engineering/claude-code-best-practices](https://www.anthropic.com/engineering/claude-code-best-practices)) — practical agentic-platform patterns; read as one host's take, not as universal law.

## Related modules

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — workflow-vs-agent judgment.
- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — subagents, MCP, and agent-to-agent interfaces at the architecture level.
- [mod-304: Evaluation Harnesses](../mod-304-evaluation-harnesses/README.md) — the eval-gating discipline that backs Chapter 4's feedback loops.
- [mod-308: Deployment & Durable Execution](../mod-308-deployment-durable-execution/README.md) — the durable human-in-the-loop gate that backs the irreversible-action gating in Chapter 2.
