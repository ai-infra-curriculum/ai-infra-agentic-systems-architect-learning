# exercise-01: Plugin Architecture on Agents, Skills, Hooks, and MCP

**Estimated effort:** 4 hours

## Objective

Design — and partially build — a plugin-based platform that extends an agentic coding tool (Claude Code as the reference) for one organization. You will take a set of platform requirements, **partition** each onto the correct extension standard (Agent, Skill, Hook, or MCP), justify every choice, and produce the real config artifacts that make the partition concrete: a `plugin.json`, one subagent, one skill, one hook, and one MCP server config. The deliverable is an architecture document plus a working plugin skeleton an engineer could fill in.

## Background

This exercise covers material from:

- [Chapter 1 — Plugin Architecture on Agents, Skills, Hooks, and MCP](../01-plugin-architecture-extension-standards.md)

The four extension points and the routing rule — **hooks for guarantees, MCP for access, skills for know-how, subagents for isolated specialized work** — are the spine of this exercise. Verify all config formats against the current Claude Code docs at [code.claude.com/docs](https://code.claude.com/docs); the structures below are illustrative and the docs are authoritative.

## Prerequisites

- Read Chapter 1 and skim the Claude Code docs sections on subagents, skills, hooks, plugins, and MCP.
- A scratch repo where you can create the plugin directory structure.
- No live model spend is required; this is a design-plus-config exercise.

## The requirements to partition

Your org (call it Acme) wants its SDLC platform to:

1. Open PRs that follow Acme's PR format (title prefix, description template, linked ticket).
2. Never write to `infra/`, `*.tf`, or `db/migrations/` without explicit human confirmation.
3. Run a focused security review on any proposed diff before a PR is finalized.
4. Read and transition Jira tickets, and read Confluence design pages.
5. Run `make lint` after every file edit and surface failures.
6. Apply Acme's house style for commit messages.

## Tasks

### 1. Partition the requirements

- For each of the six requirements, decide which extension point it belongs in (Agent / Skill / Hook / MCP), and write one sentence justifying it against the routing rule.
- Flag any requirement that decomposes into **two** extension points (e.g., a skill that *describes* plus a hook that *enforces*), and split it explicitly.
- Produce a partition table: `requirement | extension point(s) | justification`.

### 2. Write the plugin manifest

- Create `.claude-plugin/plugin.json` with a real name, semantic version, description, and author for the Acme platform plugin.

### 3. Author one subagent

- Write `agents/security-reviewer.md` with frontmatter (name, description, scoped tools) and a system prompt that reviews a diff for security issues and returns a structured verdict. Scope its tools to read-only — it inspects, it does not write.

### 4. Author one skill

- Write `skills/pr-conventions/SKILL.md` with frontmatter (name, description) and a body documenting Acme's PR title/description/ticket-link format. The description must make clear *when* the agent should load it.

### 5. Author one hook

- Write `hooks/hooks.json` with a `PreToolUse` hook that blocks edits to the protected paths from requirement 2 and a `PostToolUse` hook that runs `make lint`. Make the block deterministic — it must fire every time, not at the model's discretion.

### 6. Author one MCP server config

- Write the `.mcp.json` entry that gives the agent Jira + Confluence access (requirement 4). Reference where the credentials come from (env var), and confirm the token is held by the server, never placed in the agent's context.

## Starter guidance

A plugin manifest:

```json
{
  "name": "acme-sdlc-platform",
  "version": "1.0.0",
  "description": "Acme's internal agentic SDLC platform.",
  "author": { "name": "Developer Platform Team" }
}
```

A subagent definition (`agents/security-reviewer.md`):

```markdown
---
name: security-reviewer
description: >-
  Reviews a proposed code diff for security issues (injection, secrets,
  unsafe deserialization, authz gaps). Invoke before finalizing any PR.
tools: Read, Grep, Glob
---

You are a focused security reviewer. You see only the diff. Report findings
as: SEVERITY (critical/high/medium/low) — file:line — issue — fix. End with
a one-line verdict: BLOCK or PASS. Do not modify files.
```

A hook config (`hooks/hooks.json`):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PLUGIN_ROOT/scripts/guard-protected-paths.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "make lint" }
        ]
      }
    ]
  }
}
```

An MCP server entry (`.mcp.json`):

```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@acme/atlassian-mcp-server"],
      "env": { "ATLASSIAN_TOKEN": "${ACME_ATLASSIAN_TOKEN}" }
    }
  }
}
```

The guard script returns a non-zero exit (or a blocking JSON decision) when the edit targets a protected path; consult the hooks docs for the exact blocking contract.

## Acceptance criteria

You can demonstrate that:

- Every requirement is mapped to an extension point with a one-sentence justification, and at least one requirement is correctly split across two points.
- The partition obeys the routing rule: guarantees are hooks (not skills), access is MCP (not skills), and the protected-path block is deterministic, not model-discretionary.
- The `plugin.json`, subagent, skill, hook config, and MCP config are all present and well-formed.
- The security-reviewer subagent's tools are read-only, and no credential appears in any agent-visible context (only in the MCP server's `env`).

## Reflection

In `NOTES.md`:

1. Which requirement was hardest to place, and why? Where did "skill vs. hook" or "skill vs. MCP" tempt you toward the wrong answer?
2. Why does packaging all four extension types as one *plugin* matter for a 50-engineer org, versus letting each engineer configure their own?
3. Requirement 3 (security review) could be a subagent *or* a hard CI gate. When would you choose each, and could you want both?

## Stretch goals

- Add a marketplace entry (`.claude-plugin/marketplace.json`) so the plugin is installable org-wide, and describe the rollout/rollback story.
- Add a second subagent (`migration-writer`) with a tightly scoped toolset, and explain why it deserves its own isolated context.
- Write the `guard-protected-paths.sh` script for real, returning the correct blocking decision per the hooks docs, and test it against an edit inside and outside `infra/`.
</content>
