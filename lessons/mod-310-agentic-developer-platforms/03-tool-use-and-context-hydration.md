# Chapter 3 — Reliable Tool-Use Patterns and Context Hydration

A coding agent is only as good as the tools you hand it and the context you feed it. Give it a vague tool and it calls the wrong one with the wrong arguments. Give it the whole monorepo "for context" and it drowns — slow, expensive, and worse at the task than if you'd given it three relevant files. This chapter is about the two halves of that problem: **designing tools the model uses correctly**, and **hydrating just-enough context** so the agent has what it needs and nothing it doesn't.

## Tool design is interface design

The model decides which tool to call, and with what arguments, from one thing: the tool's **description and schema**. That description is a prompt. Treat tool design as API design for a non-deterministic consumer.

Reliable tools share these properties:

- **One clear job, named for intent.** `get_ticket(id)` and `search_code(query)` beat a god-tool `do_jira(action, params)` whose behavior depends on a stringly-typed `action`. The model picks tools by matching task to description; overloaded tools blur the match.
- **Typed, constrained inputs.** Enumerations, required fields, and bounded ranges in the schema stop whole classes of malformed calls. If a field can only be `bug | feature | chore`, make it an enum — don't accept free text and hope.
- **Descriptions that say *when*, not just *what*.** "Search the codebase by semantic query. Use this before editing to find the relevant files; do not use it for exact-symbol lookup — use `find_symbol` for that." You are routing the model away from the adjacent wrong tool.
- **Errors the model can act on.** A tool that fails should return *why* and *what to try* ("ticket not found; check the ID format `PROJ-123`"), not a stack trace. The agent reads the error and course-corrects; make that possible.
- **Bounded outputs.** A tool that can return 50,000 lines will, eventually, and it will blow the context window in one call. Paginate, truncate with a continuation token, or return a summary-plus-handle. Output size is an architectural property of the tool, not an afterthought.

A tool exposed over MCP — the integration layer from Chapter 2 — carries exactly this contract. The server author writes the description and schema; you, the architect, review them as the *interface the model programs against*.

## Context hydration: the layered strategy

"Context hydration" is the deliberate assembly of what goes into the agent's window for a task. The naive options both fail: an empty context makes the agent guess; a maximal context (whole repo, all tickets) is slow, costly, and — counterintuitively — *less accurate*, because relevant signal drowns in noise (the "context rot" failure). The architecture is a **layered, just-in-time** strategy.

```text
                       ┌─────────────────────────────┐
  always present  ───▶ │ L0  System / platform prompt │  conventions, guardrails
  (cheap, small)       │     (skill: pr-conventions)  │  — loaded by the plugin
                       ├─────────────────────────────┤
  task-scoped     ───▶ │ L1  The task itself          │  the Jira ticket, the
  (per request)        │     + acceptance criteria    │  failing test, the goal
                       ├─────────────────────────────┤
  pulled on        ──▶ │ L2  Retrieved-on-demand      │  search_code() results,
  demand (the          │     just-in-time context     │  the 3 relevant files,
  bulk, but lazy)      │     via MCP tools            │  the linked design doc
                       ├─────────────────────────────┤
  isolated /       ──▶ │ L3  Sub-agent / sub-context  │  big sub-tasks run in a
  distilled            │     returns a *summary*      │  subagent, return distilled
                       └─────────────────────────────┘
```

- **L0 — Standing context.** Conventions and guardrails that apply to every task, delivered by the plugin's skills and system prompt. Small, stable, always present. This is where "how we write PRs" lives — as a *skill* the agent loads, not as a paragraph re-pasted into every request.
- **L1 — Task context.** The specific goal: the ticket text, the acceptance criteria, the stack trace. Per-request, hydrated by the integration that kicked off the task (the Jira pull, the CI failure).
- **L2 — Retrieved-on-demand context.** The bulk of what the agent needs, but pulled **lazily** through tools, not pre-loaded. The agent calls `search_code` and reads the three files it returns — not the 3,000 files it didn't ask for. This is the same least-privilege principle from Chapter 2, applied to tokens: the model only sees what it explicitly fetched.
- **L3 — Isolated sub-context.** When a sub-task is itself large (analyze a whole module, triage 40 log lines), run it in a **subagent** with its own window and have it return a *distilled* result. The main agent's context stays clean; the heavy reading happened somewhere else and came back as a summary.

The discipline across all four layers: **pull, don't pre-load; distill, don't dump.** Every token in the window should be there because the task needs it now.

## Degrade safely when context is missing

Real SDLC context is often incomplete: the ticket is thin, the design doc is stale, the linked PR was deleted. A reliable platform makes the agent **degrade explicitly**, not hallucinate:

- **Make missing context legible.** If `get_ticket` returns a one-line ticket with no acceptance criteria, the agent should *say so* and ask, not invent requirements. Tool outputs that signal "this is sparse" let the model behave correctly.
- **Prefer asking over assuming on ambiguity.** For requirements analysis especially, an agent that asks one clarifying question beats one that confidently implements the wrong thing. Architect a path for the agent to *escalate to the human* when L1 is underspecified.
- **Fail the hydration, not silently.** If an MCP fetch errors (Jira down, repo unreachable), surface it as a tool error the agent reports — never a silent empty result the agent treats as "nothing relevant exists."

## A worked hydration plan

*"Debug this failing CI test."* The hydration plan:

| Layer | Content | Source | Budget |
| --- | --- | --- | --- |
| L0 | Debugging conventions, "don't disable tests to make them pass" guardrail | plugin skill | tiny, standing |
| L1 | The failing test name, its assertion error, the CI log tail | CI event payload | small, per-task |
| L2 | The test file + the source under test (pulled by `search_code`), the last 3 commits that touched them | MCP, on demand | bounded, lazy |
| L3 | If the failure spans many modules, a subagent traces the call path and returns a 5-line root-cause summary | subagent | distilled |

Notice what is *absent*: the rest of the repo, unrelated tickets, the full CI history. The agent gets the failing test, the code under it, recent changes, and — only if needed — a distilled trace. That plan is small, fast, and *more* accurate than a context dump, and producing one for a real task is [exercise-03](exercises/exercise-03-context-hydration-strategies.md).

## Key takeaways

- Tool descriptions and schemas are the **prompt the model programs against** — design tools with one clear intent-named job, typed inputs, actionable errors, *when*-to-use guidance, and **bounded outputs**.
- Context hydration is a **layered, just-in-time** strategy: standing conventions (L0), the task (L1), retrieved-on-demand context (L2), and distilled sub-context (L3) — **pull, don't pre-load; distill, don't dump.**
- A maximal context is not a safe default: it is slower, costlier, and *less* accurate than just-enough context because relevant signal drowns in noise.
- Architect **explicit, safe degradation** when context is missing — make sparseness legible, ask on ambiguity, and surface hydration failures instead of letting the agent hallucinate around them.
</content>
