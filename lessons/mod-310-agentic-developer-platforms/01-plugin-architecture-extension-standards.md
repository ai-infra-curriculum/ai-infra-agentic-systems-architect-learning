# Chapter 1 — Plugin Architecture on Agents, Skills, Hooks, and MCP

An agentic coding tool ships with sane defaults: a model, a tool loop, file and shell access, and a system prompt. That is enough for one engineer on one repo. The moment you want to encode *your organization's* conventions — its review standards, its deployment runbook, its internal services, its "never touch the prod migration without a human gate" rule — you are no longer a user. You are a **platform architect**, and the question becomes: through which extension point does each of those behaviors belong?

Claude Code (our concrete reference; docs at [code.claude.com/docs](https://code.claude.com/docs)) exposes four extension standards. Getting capabilities into the *right* one is the core skill of this chapter, because the wrong choice produces a platform that is slow, leaky, or unmaintainable.

## The four extension points

```text
┌──────────────────────────────────────────────────────────────┐
│                     Agentic coding tool                       │
│                                                               │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐    │
│  │  AGENTS  │   │  SKILLS  │   │  HOOKS   │   │   MCP    │    │
│  │(subagent)│   │(procedur-│   │(determin-│   │(external │    │
│  │ isolated │   │  al know- │   │  istic   │   │  tools / │    │
│  │ context, │   │  how, on-│   │ lifecycle│   │ data over│    │
│  │ own tools│   │  demand) │   │  events) │   │  a server│    │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘    │
│       │              │              │              │          │
│       └──── model-invoked ──────────┘   └─ shell ──┘          │
│                                    deterministic   protocol   │
└──────────────────────────────────────────────────────────────┘
```

The axis that organizes them: **who decides when this runs, and is it deterministic?**

- **Agents (subagents)** — a separate agent with its own context window, its own system prompt, and a scoped tool set. The main agent *delegates* to it. Use a subagent when a task deserves an isolated context and a specialized persona (a `security-reviewer` that only sees the diff, a `migration-writer` with a tight toolset). The model decides when to invoke it, based on the subagent's description.
- **Skills** — packaged procedural knowledge: a `SKILL.md` plus optional scripts and references that the agent loads **on demand** when the task matches. A skill is "how we do X here" — your PR-description format, your incident-runbook steps, your codegen conventions. It is model-invoked and progressive: the agent reads the short description first, pulls in the body only when relevant. This keeps the base context lean.
- **Hooks** — shell commands the tool runs **deterministically** on lifecycle events (`PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`, and others). Hooks are *not* model-invoked — they fire every time the event fires, by configuration, not by the model's choice. This is where you put guarantees: format-on-edit, block-writes-to-protected-paths, run-the-linter-before-stop. If a behavior must happen *every time* and cannot be left to the model's discretion, it is a hook.
- **MCP (Model Context Protocol)** — an open protocol (spec at [modelcontextprotocol.io](https://modelcontextprotocol.io)) for connecting the agent to **external tools, resources, and prompts** served by a separate process. An MCP server is how the agent reaches your internal services, your ticketing system, your database — anything outside the local filesystem and shell. The model invokes MCP tools like any other tool, but they are implemented and governed *outside* the agent.

## The decision rule

When a new capability lands on your desk, route it:

| If the capability is… | …it belongs in a |
| --- | --- |
| A specialized task deserving its own clean context and persona | **Agent / subagent** |
| Procedural know-how the agent should apply when the task matches | **Skill** |
| A guarantee that must run on an event, every time, deterministically | **Hook** |
| Access to an external system, dataset, or service | **MCP server** |

The traps are the overlaps:

- **"Run the linter."** If you want the agent to lint *when it thinks it should*, that is a skill. If you want lint to run on *every* file write no matter what, that is a hook. Same capability, opposite extension point — chosen by whether you need a guarantee or a suggestion.
- **"Reach the internal deployment service."** That is an MCP tool, not a skill. The skill might *describe how to deploy*; the MCP server is *how the agent actually calls deploy*. Procedural knowledge and system access are different layers and belong in different extension points.
- **"Review security on every PR."** A `security-reviewer` subagent gives you isolation and a focused persona — but if you need a *hard block* on merging without review, the block is a hook (or a CI gate), not a subagent the model may or may not invoke.

The principle: **hooks for guarantees, MCP for access, skills for know-how, subagents for isolated specialized work.** Capabilities that feel like they could go anywhere usually decompose into two: a skill that *describes* and an MCP tool or hook that *enforces*.

## Packaging it as a plugin

A handful of agents, skills, hooks, and MCP server configs scattered across engineers' machines is not a platform — it is configuration drift. Claude Code's **plugin** format packages these four extension types into one distributable, versioned unit your org installs from a marketplace (a Git repo of plugins). A plugin directory bundles:

```text
my-org-platform/
├── .claude-plugin/
│   └── plugin.json          # name, version, description, author
├── agents/                  # subagent definitions (Markdown + frontmatter)
│   ├── security-reviewer.md
│   └── migration-writer.md
├── skills/                  # SKILL.md packages
│   └── pr-conventions/
│       └── SKILL.md
├── hooks/
│   └── hooks.json           # lifecycle event → command mappings
└── .mcp.json                # MCP servers this plugin provides
```

This packaging is the architectural payoff. One install gives every engineer the same subagents, the same conventions, the same guarantees, and the same internal-service access — versioned, reviewable in a PR, and rolled out (or rolled back) centrally. The platform team owns the plugin; engineers consume it; drift collapses to one source of truth.

A minimal `plugin.json`:

```json
{
  "name": "acme-sdlc-platform",
  "version": "1.4.0",
  "description": "Acme's internal agentic SDLC platform: review subagents, PR conventions, protected-path guards, and internal-service MCP access.",
  "author": { "name": "Developer Platform Team" }
}
```

## A worked partition

Take one real requirement: *"Engineers should be able to ask the agent to implement a Jira ticket, and the result must follow our PR format, never write to `infra/` without a human gate, and be reviewed for security before it's proposed."* Partition it across the four extension points:

- **MCP server** — `jira` access (read the ticket, its description, acceptance criteria) and `github` access (open the PR). System access → MCP.
- **Skill** — `pr-conventions`: how Acme writes PR titles, descriptions, and links tickets. Know-how the agent applies when opening a PR.
- **Hook** — `PreToolUse` on file writes: block any edit under `infra/` and require explicit human confirmation. A guarantee that must hold every time → hook.
- **Agent** — `security-reviewer` subagent: runs on the proposed diff in an isolated context before the PR is finalized. Specialized, isolated work → subagent.

One requirement, cleanly decomposed, with each piece in the extension point that fits its semantics. That decomposition *is* the architecture — and it is exactly the deliverable in [exercise-01](exercises/exercise-01-plugin-architecture-agents-skills-hooks-mcp.md).

## Key takeaways

- The four extension standards split on **who triggers them and whether it's deterministic**: agents and skills are model-invoked; hooks are deterministic lifecycle events; MCP is external access invoked as tools.
- Route capabilities with one rule: **hooks for guarantees, MCP for access, skills for know-how, subagents for isolated specialized work.** Apparent overlaps usually decompose into a "describe" skill plus an "enforce" hook or "access" MCP tool.
- A **plugin** packages all four into one versioned, installable unit — turning ad-hoc per-engineer config into a centrally owned platform with a single source of truth.
- The architect's deliverable is the *partition*: mapping each requirement onto the correct extension point, which is what makes the platform fast, safe, and maintainable instead of leaky and drift-prone.
</content>
