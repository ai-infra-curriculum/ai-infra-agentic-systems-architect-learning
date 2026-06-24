# Resources for mod-310-agentic-developer-platforms (Agentic Developer Platforms & SDLC Integration)

Primary references for designing an agentic developer platform on an extensible coding tool. Verify against current docs — agent tooling and platform APIs move fast.

## Claude Code extension standards (the four extension points)

Docs live at [code.claude.com/docs](https://code.claude.com/docs). These are the concrete reference for Chapter 1's plugin architecture.

- **Subagents** ([code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)) — isolated-context agents with their own system prompt and scoped tools; the "Agents" extension point.
- **Skills / Agent Skills** ([code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)) — packaged procedural knowledge (`SKILL.md`) loaded on demand; see also the engineering write-up *Equipping agents for the real world with Agent Skills* ([anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).
- **Hooks** ([code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks)) — deterministic shell commands on lifecycle events (`PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`, …); the guarantees layer.
- **MCP in Claude Code** ([code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp)) — connecting external tools/resources via MCP servers; the access layer.
- **Plugins** ([code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins)) and **plugin marketplaces** ([code.claude.com/docs/en/plugin-marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)) — packaging all four extension types into one versioned, installable unit.

## Model Context Protocol

- **MCP specification and docs** ([modelcontextprotocol.io](https://modelcontextprotocol.io)) — the open protocol for tools, resources, and prompts. The authoritative reference for the access layer.
- **MCP servers reference** ([github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)) — reference and community server implementations to study before building your own.

## AI for the SDLC and effective agents

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — workflow-vs-agent judgment and the orchestration patterns underpinning a platform's internals.
- **Anthropic — Claude Code best practices** ([anthropic.com/engineering/claude-code-best-practices](https://www.anthropic.com/engineering/claude-code-best-practices)) — practical patterns for agentic coding workflows that inform tool design and context hydration.
- **Claude Agent SDK** ([code.claude.com/docs/en/sdk](https://code.claude.com/docs/en/sdk)) — for building platform services around the agent loop with subagent isolation and tool scoping.

## DevOps and PM toolchain APIs (Chapter 4)

- **GitHub REST API** ([docs.github.com/en/rest](https://docs.github.com/en/rest)) — writes (open/update PR, comment) and shallow reads.
- **GitHub GraphQL API** ([docs.github.com/en/graphql](https://docs.github.com/en/graphql)) — deep/related reads in one round-trip; note the point-based rate budget.
- **GitHub webhooks** ([docs.github.com/en/webhooks](https://docs.github.com/en/webhooks)) — event-driven triggers (PR opened, check completed, review submitted); see also signature verification.
- **Jira Cloud REST API** ([developer.atlassian.com/cloud/jira/platform/rest/v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)) — read/transition issues, comment.
- **Jira webhooks** ([developer.atlassian.com/cloud/jira/platform/webhooks](https://developer.atlassian.com/cloud/jira/platform/webhooks/)) — react to issue events.
- **Confluence Cloud REST API** ([developer.atlassian.com/cloud/confluence/rest/v2](https://developer.atlassian.com/cloud/confluence/rest/v2/)) — read design/spec pages for requirements context.

## Security and trust boundaries (Chapter 2)

- **OWASP Top 10 for LLM Applications** ([owasp.org/www-project-top-10-for-large-language-model-applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)) — prompt injection, insecure tool use, excessive agency; the threat catalog for the trust-boundary spec.

## Related modules

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — workflow-vs-agent judgment.
- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — subagents, MCP, A2A at the architecture level.
- [mod-304: Evaluation Harnesses](../mod-304-evaluation-harnesses/README.md) — the eval-gating discipline that backs Chapter 5's feedback loops.
</content>
