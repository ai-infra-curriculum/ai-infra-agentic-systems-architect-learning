# exercise-05: Non-SDLC Platform Case

**Estimated effort:** 2 hours

## Objective

Take the platform pattern you designed for a software-development context across exercises 01–04 and **re-bind it to a non-developer internal domain** — ops, support, or analytics — to prove the architecture is domain-independent. The deliverable is a side-by-side mapping showing what *changed* (the nouns: inputs, tools, systems, irreversible-action list) and what *stayed constant* (the architecture: extension model, trust asymmetry, hydration, interface selection, DX), with a one-sentence verdict on generalization.

This is an **architecture** exercise, and the point is the proof, not the volume. A tight, honest mapping beats a sprawling redesign.

## Background

This exercise covers material from:

- [Chapter 5 — Beyond SDLC: Generalizing the Platform](../05-beyond-sdlc-generalizing-the-platform.md)
- All four prior chapters — this exercise re-uses their architecture wholesale.

## The scenario

You built a strong agentic platform for a development context (exercises 01–04). Leadership now wants an agent platform for a **non-developer internal domain**. Pick **one**:

> **(a) Operations / on-call** — triage incidents against monitoring, logs, paging, and the status page.
>
> **(b) Customer support** — resolve tickets against customer records, the ticketing system, and a knowledge base, with refunds/escalation as actions.
>
> **(c) Analytics / data** — answer internal questions against a data warehouse, publish dashboards, and run scoped queries.

The question to answer: **do we start over, or re-bind?** Your job is to prove it is a re-bind by walking the five pattern elements.

## Tasks

### 1. Restate the platform pattern domain-free

- Write the five-element pattern from [Chapter 5](../05-beyond-sdlc-generalizing-the-platform.md) (extension model / trust asymmetry / hydration / integration / adoption+DX) with **no domain-specific words** — no code, commits, or developers.

### 2. Re-populate the nouns for your chosen domain

- **Extension model** — name the *agents*, *skills*, *tools*, *hooks*, and *MCP servers* for this domain. Apply the same placement rule (guarantee → hook, external access → MCP, know-how → skill, callable → tool, role → agent).
- **Trust asymmetry** — list this domain's **untrusted inputs** (e.g., log lines, customer messages, raw datasets) and its **privileged tools**, then build the domain's **reversibility ladder** with the irreversible tier gated.

| Element | This domain's nouns |
| --- | --- |
| Untrusted inputs | |
| Privileged tools | |
| Irreversible (gated) actions | |
| External systems (MCP) | |

### 3. Confirm hydration and integration carry over unchanged

- Lay out the four hydration layers for this domain (standing / task / retrieved / tool-result) and confirm "pull, don't pre-load; distill, don't dump" holds — with one concrete distillation from this domain.
- Apply the interface-selection rule (REST / GraphQL / event) to this domain's systems, and confirm the reactive trigger (new alert / new ticket / new request) is an **event**, not a poll.

### 4. Build the changed-vs-constant table

- Produce the side-by-side table from [Chapter 5](../05-beyond-sdlc-generalizing-the-platform.md): rows are the pattern elements, columns are the SDLC binding, your non-SDLC binding, and a verdict column that should read **"same"** for the architecture.

| Element | SDLC binding | Your non-SDLC binding | Architecture |
| --- | --- | --- | --- |

### 5. Render the verdict

- Answer the leadership question in one sentence: **re-bind, not rebuild** — and state precisely what you *re-author* (the noun layer: domain MCP servers, domain skills, the re-populated irreversible-action list) versus what carries over wholesale (the trust boundaries, gating discipline, eval loop, DX).

## Starter guidance

A pattern-restatement to react to:

```text
   1 EXTENSION MODEL   agents / skills+tools / hooks / MCP, by capability KIND
   2 TRUST ASYMMETRY   inputs untrusted, tools privileged; gate irreversible
   3 HYDRATION         pull+distill, layered, degrade safely
   4 INTEGRATION       REST writes / GraphQL deep reads / events react
   5 ADOPTION + DX     activation + trust; docs pay twice; eval-gated feedback
```

An ops re-population to adapt (if you pick (a)):

```text
   untrusted inputs : alerts, log lines, customer messages
   privileged tools : restart service, page, post status, refund
   irreversible/gated: restart prod, public status post, refund, page exec
   external systems : monitoring, logs, paging, ticketing, status page (each MCP)
```

## Acceptance criteria

You can demonstrate that:

- The five-element pattern is restated with **no domain-specific vocabulary**.
- The chosen domain's nouns are fully re-populated — agents/skills/tools/hooks/MCP, untrusted inputs, privileged tools, gated irreversible actions, external systems.
- Hydration and integration are shown to carry over unchanged, with one concrete distillation and the reactive trigger correctly designed as an event.
- The changed-vs-constant table reads **"same"** down the architecture column, with only the nouns differing.
- The verdict is **re-bind, not rebuild**, with the re-authored noun layer and the carried-over architecture named explicitly.

## Reflection

In `NOTES.md`:

1. Which pattern element felt *least* obviously domain-independent before you mapped it, and what convinced you it carried over?
2. Where, if anywhere, did the non-SDLC domain genuinely need something the SDLC binding did not — and is that a new architecture element, or just a new noun?
3. If you had built the SDLC platform with no architecture (just a wired-up coding tool), what specifically would force a rebuild here that the pattern-based design avoids?

## Stretch goals

- Map a **second** non-SDLC domain (e.g., analytics if you did ops) and show the architecture column is *still* "same" — two re-bindings is a stronger proof than one.
- Identify the **one** place across all five elements where this domain's stakes are *higher* than SDLC's (e.g., an irreversible ops action during an incident) and tighten that gate accordingly.
- Sketch a single **shared platform core** (host-agnostic MCP servers + the trust/eval discipline) that multiple domain bindings could sit on top of, and name what lives in the core versus each domain binding.
