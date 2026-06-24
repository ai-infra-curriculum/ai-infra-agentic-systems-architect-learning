# Chapter 1 — The Extension Model Across Platforms

Every agentic platform worth building is, at its core, a base agent loop plus a way to **extend** it: give it new behaviors, new knowledge, new guarantees, and new reach into external systems. The vendors who ship these platforms each invented their own names and packaging for those extensions, and it is tempting to learn one vendor's vocabulary and call it "the way agent platforms work." That is a trap. The vocabulary is host-specific; the **architecture is not**. Underneath the brand names there are four recurring extension points, and the architect's job is to design at the level of those four — then bind the design to whatever host the org actually runs, knowing exactly how portable that binding is.

This chapter establishes the generic extension model, maps it across named platforms **as peers**, and teaches you to grade portability and lock-in — because choosing a host is a reversible decision only if you designed for it to be.

## The four generic extension points

Strip away the branding and every extensible agent host gives you some version of these four. They are different *kinds* of thing, and the single most common architecture mistake is putting a capability in the wrong one.

```text
   ┌──────────────────────────────────────────────────────────────────┐
   │  AGENTS (sub-agents / roles)                                       │
   │    a scoped, separately-prompted agent with its own tools and      │
   │    context window — for decomposing work and isolating context     │
   ├──────────────────────────────────────────────────────────────────┤
   │  SKILLS / TOOLS (procedural knowledge + callable functions)        │
   │    packaged know-how the agent loads on demand, and the local       │
   │    functions it can invoke — "how to do X" and "do X"               │
   ├──────────────────────────────────────────────────────────────────┤
   │  HOOKS / LIFECYCLE EVENTS (deterministic interception)             │
   │    code that runs on a lifecycle event (before/after a tool call,   │
   │    on session start, on stop) — guarantees, not suggestions         │
   ├──────────────────────────────────────────────────────────────────┤
   │  MCP (external access)                                              │
   │    a standard protocol binding the agent to external tools,         │
   │    resources, and prompts living outside the host                   │
   └──────────────────────────────────────────────────────────────────┘
```

The distinctions that matter:

- **Agents** are about *decomposition and context isolation*. A reviewer sub-agent with a tight prompt and read-only tools keeps the main agent's context clean and bounds what that role can do. The unit of design is "what is a separate role, with what tools, seeing what context."
- **Skills/tools** split into two: **skills** are *procedural knowledge* ("how we cut a release," "how we write a postmortem") the agent pulls in when relevant; **tools** are *callable functions* the agent invokes. The line: a skill *describes*; a tool *does*. Confusing the two is the canonical error — "reach the deploy service" is a tool (access), not a skill (know-how), even though a skill might describe *when* to deploy.
- **Hooks/lifecycle events** are about **guarantees**. A model may or may not follow an instruction in a skill. A hook on the "before tool call" event runs *every* time, deterministically, regardless of what the model decided. If a behavior must happen on every event — lint on every write, block a forbidden command, log every action — it is a hook, never a skill.
- **MCP** is about **reach across a boundary**. The Model Context Protocol is the open standard for connecting an agent to tools, resources, and prompts that live *outside* the host process — your internal services, your databases, your SaaS APIs. Access to an external system is MCP's job, not a skill's or a hook's.

The decision rule, stated once:

> **Guarantee → hook. External access → MCP. Procedural know-how → skill. Callable local function → tool. Separate role / isolated context → agent.** Choose by the *kind* of capability, not by which one is easiest to write.

## Mapping the model across platforms (as peers)

No platform is *the* platform. Below, the four extension points are mapped across five hosts treated as equals — three commercial agentic coding tools, one orchestration platform, and the always-available option of a custom in-house host. Read it as: "the same architecture, bound five different ways."

| Generic point | Claude Code | Cursor | GitHub Copilot | LangGraph Platform | Custom host |
| --- | --- | --- | --- | --- | --- |
| **Agents** | Subagents (scoped prompt + tools) | Custom modes / agent profiles | Coding agent + chat participants | Graph nodes / sub-graphs as agents | Your own agent objects + loop |
| **Skills** | Agent Skills (`SKILL.md`, loaded on demand) | Rules / `.cursor/rules`, docs context | Custom instructions, repo instructions | Prompt templates + retrieved docs | Your prompt/knowledge assembly |
| **Tools** | Built-in tools + MCP tools | Built-in edit/run tools + MCP | Built-in tools + extensions | Python/JS functions bound as tools | Functions you register |
| **Hooks / lifecycle** | Hooks (`PreToolUse`, `Stop`, …) | Limited lifecycle hooks | Limited; CI/policy as the gate | Graph edges, interrupts, callbacks | Whatever events you emit |
| **MCP (external)** | First-class MCP client | MCP client support | MCP support (evolving) | MCP client + native tool nodes | You implement the MCP client |

Three things to read out of this table.

First, **the columns are not uniform** — and that asymmetry is the whole point. Every host gives you *agents* and *tools* and *MCP* in some form, because those are table stakes. But the **hooks/lifecycle** row is where hosts diverge sharply: Claude Code and LangGraph give you rich, deterministic interception points; a coding tool with a thinner lifecycle pushes you to enforce the same guarantee *outside* the agent (in CI, in a policy gate, in a server you front the agent with). The architect's move is to notice which guarantees a host can enforce in-loop versus which you must enforce out-of-loop, and design accordingly.

Second, **the custom-host column is always live.** Treating "build your own host" as a peer keeps you honest: any capability you can only get from a vendor's proprietary packaging is a lock-in cost you are choosing to pay. Sometimes paying it is right (you get the packaging for free); sometimes it is not (you could implement the same MCP binding yourself and stay portable).

Third, **MCP is the portability anchor.** Because MCP is an open protocol rather than a vendor feature, the external-access layer is the *most* portable part of the whole architecture. An MCP server you write for your internal deploy system works against any MCP-capable host. The integration logic — the part that is expensive to build and dangerous to get wrong — is reusable. That is a deliberate architectural lever: push as much of your platform's value into MCP servers as you reasonably can, and the host underneath becomes swappable.

## Packaging: from scattered config to a distributable unit

Extension points are useless to an org if every engineer wires them up by hand. The four points must be **packaged** into a single, versioned, installable unit — a *plugin* or *bundle* — so that "install the platform" is one action and config stops drifting across machines.

```text
   platform-bundle/
     agents/        ← the roles (reviewer, debugger, triager, …)
     skills/        ← procedural know-how (release process, runbooks, …)
     tools/         ← local callable functions
     hooks/         ← lifecycle guarantees (lint, policy, audit)
     mcp/           ← external-access server configs (internal services)
     manifest       ← version, host-binding, dependencies
```

What packaging buys a 50-person org:

- **One source of truth.** The bundle is versioned in a repo; "everyone is on v2.3" is a checkable fact, not a hope.
- **Reviewable change.** A new hook or a widened tool scope is a pull request, with the security team in the review path.
- **Reproducible onboarding.** A new hire installs the bundle and has the org's agents, skills, hooks, and MCP servers — not a blank tool they must configure correctly from memory.

Hosts package differently: Claude Code has plugins and plugin marketplaces; other hosts use settings files, extensions, or a deployed service. The **packaging mechanism is host-specific; the contents are the portable part.** Design the contents as four clean folders mapped to the four extension points, and re-binding to a new host's packaging format is a contained job rather than a rewrite.

## Grading portability and lock-in

Choosing a host is only a reversible decision if you measured its reversibility up front. For each capability in your platform, grade two things.

**Portability** — how much of this survives a host switch unchanged?

- *High* — an MCP server (open protocol; rebinds to any MCP host), a tool's business logic, the *contents* of a skill as prose.
- *Medium* — agent role definitions and prompts (concepts transfer; format and tool-binding syntax do not), context-hydration logic.
- *Low* — anything expressed in a host's proprietary hook/lifecycle format, host-specific packaging manifests, features with no equivalent elsewhere.

**Lock-in** — what does this host capture that you cannot easily reproduce?

```text
   lock-in surface per capability
   ────────────────────────────────────────────────
   external access (MCP)     ▏ low      (open protocol, portable)
   tool/business logic       ▏ low      (your code, host-agnostic)
   skills (as prose)         ▏ low–med  (content portable; loading host-specific)
   agent/role definitions    ▏ medium   (concept portable; binding is not)
   hooks / lifecycle         ▏ high     (proprietary event model)
   proprietary-only features ▏ high     (no equivalent off-host)
```

The architect's lock-in register, per capability: *which extension point, on which host, how portable, and what is the exit cost if we leave this host?* A platform built mostly of MCP servers and host-agnostic tool logic, with a thin host-specific layer of hooks and packaging, is **cheap to re-host**. A platform whose core guarantees live in one vendor's proprietary lifecycle model is **expensive to leave** — which may be an acceptable trade, but only if it was chosen, not stumbled into.

## Worked judgment

*"We must run our security linter on every single file the agent writes, no exceptions."* — This is a **guarantee on a lifecycle event**, so it is a **hook** (a `PreToolUse`/`PostToolUse`-style interception) on a host that has rich hooks. On a host with a thin lifecycle, you enforce the same guarantee **out-of-loop** in CI or a policy gate fronting the agent. Either way it is *never* a skill — a skill the model "should" follow is not a guarantee. **Portability: low** (proprietary lifecycle); plan the out-of-loop fallback so a host switch does not silently drop the guarantee.

*"The agent needs to read and update tickets in our internal issue tracker."* — This is **external access**, so it is an **MCP server** wrapping the tracker's API. **Portability: high** — the same MCP server works against any MCP-capable host, so this capability does not lock you to a vendor at all. Push value here on purpose.

*"Codify how our team writes a postmortem so the agent follows it."* — Procedural know-how → a **skill** (prose the agent loads on demand). The *content* is fully portable; only the loading mechanism is host-specific.

## Key takeaways

- Under every host's branding are **four generic extension points**: agents (decomposition/isolation), skills/tools (know-how + callable functions), hooks/lifecycle (deterministic guarantees), and MCP (external access). Design at this level, not at one vendor's vocabulary.
- The decision rule: **guarantee → hook; external access → MCP; procedural know-how → skill; callable function → tool; isolated role → agent.** Putting a capability in the wrong point is the canonical platform error.
- Map the model across hosts **as peers** — Claude Code, Cursor, Copilot, LangGraph, a custom host. The columns are *not* uniform; the **hooks/lifecycle** row diverges most, and **MCP is the portability anchor** because it is an open protocol.
- **Package** the four points into one versioned bundle so config stops drifting; the *contents* are portable, the *packaging mechanism* is host-specific.
- **Grade portability and lock-in per capability.** A platform built mostly of MCP servers and host-agnostic logic is cheap to re-host; one whose guarantees live in a proprietary lifecycle is expensive to leave. Choose the trade — never stumble into it.
