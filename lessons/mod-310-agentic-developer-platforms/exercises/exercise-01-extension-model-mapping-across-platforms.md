# exercise-01: Extension-Model Mapping Across Platforms

**Estimated effort:** 4 hours

## Objective

Take a single internal agent platform's capabilities and **map the same extension architecture onto two or more named host platforms as peers** — then grade the portability and lock-in of each binding. The deliverable is a capability-to-extension-point design, a cross-platform mapping table, and a portability/lock-in register an architect could use to defend a host choice (or a deliberate multi-host hedge) to a security and platform team.

This is an **architecture** exercise. You will not build a working plugin on any host; you will produce the mapping, the grading, and the justification. Real config snippets are welcome where they make a binding concrete; a running platform is not the goal.

## Background

This exercise covers material from:

- [Chapter 1 — The Extension Model Across Platforms](../01-extension-model-across-platforms.md)

Assume your team can already build an MCP server, write a hook, and define a tool on a specific host (the L30 agentic-ai-engineer skill). Your job is the layer above: deciding which extension point each capability belongs in, how that binds across hosts, and how much each binding locks you in.

## The scenario

Your org wants an internal agent platform with this capability set:

> 1. A **reviewer** role that examines proposed work with read-only access and a tight prompt.
> 2. A **security linter that must run on every single artifact the agent produces** — no exceptions, no model discretion.
> 3. Access to an **internal service** (e.g., a deploy system or an internal API) the agent must call.
> 4. The **org's convention for a recurring procedure** (e.g., how you cut a release, or how you write a customer reply) that the agent should follow.
> 5. A **local helper function** that transforms data the agent already has (no external call).

Pick **at least two named host platforms to treat as peers** from: Claude Code, Cursor, GitHub Copilot, LangGraph Platform, or a **custom in-house host**. At least one of your two must be either a custom host *or* a platform other than Claude Code — Claude Code may be one peer, never the only or canonical one.

## Tasks

### 1. Place each capability on the generic extension model

- For each of the five capabilities, assign it to one of the four generic extension points — **agent**, **skill**, **tool**, or **hook/lifecycle**, with **MCP** as the external-access mechanism — per [Chapter 1](../01-extension-model-across-platforms.md).
- Justify each placement by the *kind* of capability (guarantee → hook; external access → MCP; procedural know-how → skill; callable function → tool; isolated role → agent).
- Capability 2 (the always-run linter) and capability 3 (the internal service) are the two most-commonly-misplaced. State explicitly why 2 is **not** a skill and why 3 is **not** a skill.

| Capability | Generic extension point | Why this point (not the others) |
| --- | --- | --- |

### 2. Map the architecture across two+ named platforms

- Build a mapping table: rows are the five capabilities, columns are your two-or-more chosen host platforms (named). Each cell names that host's concrete mechanism for the capability.

| Capability | Generic point | Platform A (named) | Platform B (named) | Platform C (optional) |
| --- | --- | --- | --- | --- |

- Call out at least one cell where a host has **no clean equivalent** for an extension point (most likely in the hooks/lifecycle row) and state how you would enforce that guarantee **out-of-loop** instead (CI, a policy gate, a fronting service).

### 3. Grade portability per capability

- For each capability, grade **portability** — High / Medium / Low — across your chosen hosts, with a one-line reason, per [Chapter 1](../01-extension-model-across-platforms.md).
- Your MCP-backed capability (the internal service) should grade **High**; your hook/lifecycle capability should grade **Low**. If your grades disagree with that, defend why.

| Capability | Portability (H/M/L) | What survives a host switch unchanged | What must be re-authored |
| --- | --- | --- | --- |

### 4. Build the lock-in register

- For each capability, record what the host *captures* that you could not easily reproduce, and the **exit cost** if you left that host.

| Capability | Host | Lock-in surface | Exit cost if we leave this host |
| --- | --- | --- | --- |

- Conclude with one sentence: is this platform, as designed, **cheap to re-host** or **expensive to leave** — and is that the trade you intend?

### 5. Defend a host decision

- Using your tables, make a recommendation: a single host, or a deliberate split (e.g., MCP servers host-agnostic, hooks on a rich-lifecycle host). Argue it against at least one rejected alternative — do not assert it.

## Starter guidance

A placement sketch to react to (refine it; do not adopt it uncritically):

```text
   capability                          generic point
   ─────────────────────────────────   ─────────────
   1 reviewer (read-only, tight prompt) → AGENT
   2 always-run security linter         → HOOK (lifecycle)   ← NOT a skill
   3 internal service access            → MCP (external)     ← NOT a skill
   4 release/reply convention           → SKILL (know-how)
   5 local data-transform helper        → TOOL (callable fn)
```

A portability/lock-in shape to adapt:

```text
   capability        portability   lock-in     note
   ────────────────  ───────────   ─────────   ─────────────────────────
   3 MCP service     HIGH          low         open protocol → any host
   2 always-run hook LOW           high        proprietary lifecycle;
                                               plan out-of-loop CI fallback
```

You do **not** need the secure tool-use design (exercise-02) or the integration interfaces (exercise-03) here — note where capability 3's MCP server hands off to those, but design them there.

## Acceptance criteria

You can demonstrate that:

- All five capabilities are placed on the generic extension model with a *kind*-based justification, and the two misplacement traps (always-run linter, internal-service access) are explicitly defended as hook and MCP respectively — not skills.
- The mapping table names **two or more host platforms as peers**, Claude Code is at most one of them, and at least one cell with no clean host equivalent is identified with an out-of-loop fallback.
- Portability is graded per capability with the MCP capability High and the hook capability Low (or a defended exception).
- The lock-in register gives an exit cost per capability and a one-sentence cheap-to-re-host vs. expensive-to-leave verdict.
- The host recommendation is argued against a rejected alternative, not asserted.

## Reflection

In `NOTES.md`:

1. Which capability was hardest to place on the generic model, and what tipped your decision?
2. For the host with the weakest hooks/lifecycle, trace exactly what breaks if you *tried* to enforce the always-run linter in-loop anyway, and how your out-of-loop fallback closes the gap.
3. If your org mandated a host switch next quarter, which capability would hurt the most to re-author, and what design change now would reduce that pain?

## Stretch goals

- Add a **fourth host** (custom in-house) as a column and estimate the build cost of reproducing each extension point yourself versus inheriting it from a vendor.
- Design the **packaged bundle** layout (the four-folder structure from [Chapter 1](../01-extension-model-across-platforms.md)) for this platform, and show which folders are host-portable and which need a per-host packaging manifest.
- Pick one capability and write the **actual MCP server tool definition** (or hook config) for one host, to ground the mapping in real config.
