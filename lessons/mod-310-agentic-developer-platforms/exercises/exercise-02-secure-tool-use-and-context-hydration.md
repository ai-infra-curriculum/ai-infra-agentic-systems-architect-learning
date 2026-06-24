# exercise-02: Secure Tool-Use and Context Hydration

**Estimated effort:** 4 hours

## Objective

Design the **secure tool-use and context-hydration architecture** for an agent that acts on internal, proprietary data: a scoped least-privilege tool set sorted by reversibility, a credentials/trust-boundary placement, a layered hydration strategy with a defended context budget, and a safe-degradation plan for missing context. The deliverable is a design artifact — a trust-boundary diagram, a tool/privilege register, a hydration layer plan, and a degradation table — that an engineer could implement on any host without re-deciding any of this.

This is an **architecture** exercise. You will not build a working agent; you will produce diagrams, registers, and tables. Pseudocode is welcome where it clarifies a boundary.

## Background

This exercise covers material from:

- [Chapter 2 — Secure Tool-Use and Context Hydration](../02-secure-tool-use-and-context-hydration.md)

Assume your team can already define a tool and wire an MCP server (the L30 skill). Your job is the layer above: where the trust boundary sits, how privilege is scoped, and how context is hydrated and degraded.

## The scenario

Design the secure interface and hydration for an agent that **acts on internal data**. Pick **one** of these (or propose an equivalent non-trivial internal-data scenario):

> **(a) SDLC binding** — an agent that reads tickets, reads private source, proposes code changes, runs tests in a sandbox, and can open (but not merge) a change.
>
> **(b) Non-SDLC binding** — a support agent that reads customer records and tickets, drafts replies, and can issue small refunds or escalate.

Either way the agent reads **untrusted internal inputs** (ticket text, file content, customer messages — any of which could carry an injected instruction) and holds **privileged tools** over internal systems. That is the trust asymmetry the design must defend.

## Tasks

### 1. Draw the trust boundary

- Produce a diagram placing untrusted inputs, the agent's reasoning context, and the privileged tools, marking explicitly where the **trust boundary** is — and stating the assumption that **the model is not itself a boundary**, per [Chapter 2](../02-secure-tool-use-and-context-hydration.md).
- Show the prompt-injection path: untrusted text → context → a privileged action, and where your design interrupts it.

### 2. Build the tool/privilege register

- Enumerate every tool the agent needs. For each, scope the **target** (not just the verb), classify it on the **reversibility ladder** (read / sandbox-write / irreversible), and state the **gate** (free, free, or human/policy gate).

| Tool | Scoped target | Reversibility tier | Gate | Where the credential lives |
| --- | --- | --- | --- | --- |

- At least one tool must be irreversible and therefore gated. State the credential placement rule (integration/MCP layer, never the model's context) and confirm no tool violates it.

### 3. Design the context-hydration layers

- Lay out the four hydration layers (standing / task / retrieved / tool-result) for this scenario, naming what goes in each and assigning each a **budget** (token ceiling or equivalent), per [Chapter 2](../02-secure-tool-use-and-context-hydration.md).
- For the **retrieved** layer, show one concrete distillation: a large internal artifact (a long spec, a long incident, a long record) reduced to the slice the task needs — not pasted whole.
- Justify why "load everything for safety" is rejected here on **all three** axes (cost, latency, *and accuracy* / context rot).

| Layer | What's in it (this scenario) | Budget | Fetched or retained? |
| --- | --- | --- | --- |

### 4. Build the safe-degradation table

- Enumerate the ways context can be missing/partial/stale (knowledge base down, ticket has no description, file deleted, search empty, record not found). For each, specify the **safe** behavior.

| Missing context | Safe degradation | What is forbidden |
| --- | --- | --- |

- At least one row must show missing context **promoting an action up the reversibility ladder** into a human gate. Every row's "forbidden" column must rule out **fabricating** the missing context.

## Starter guidance

A trust-boundary shape to react to:

```text
   UNTRUSTED                        PRIVILEGED
   ticket / record / message ─┐    ┌─▶ irreversible (GATE)
   file / log content ────────┼──▶ │   sandbox write (free)
                              │    └─▶ read (free, scoped)
              model is NOT a boundary; gate sits on the tool layer
```

A reversibility-sort to adapt:

```text
   read internal data            → allow, scoped to need
   write to sandbox/branch/draft → allow
   IRREVERSIBLE (merge/deploy/    → GATE: model proposes, human/policy disposes
     refund/prod write/send)
```

A hydration budget to refine:

```text
   L1 standing  : role + rules            (small, always on)
   L2 task      : THIS ticket/record/file (scoped to goal)
   L3 retrieved : distilled related slice (pulled on demand)
   L4 tool-result: last output, pruned after use
```

You do **not** need the toolchain interface choices (exercise-03) here — note where a tool's MCP server would bind to a real system, but choose the interface there.

## Acceptance criteria

You can demonstrate that:

- The trust boundary is drawn with the model explicitly *not* treated as a boundary, and the injection path is shown and interrupted.
- Every tool is scoped by target, placed on the reversibility ladder, and the irreversible tier is gated; **no credential lives in the model's context**.
- The four hydration layers each have a budget, the retrieved layer shows a concrete distillation, and "load everything" is rejected on cost, latency, **and accuracy**.
- The degradation table covers missing/partial/stale context, never permits fabrication, and at least one row promotes an action into a gate.

## Reflection

In `NOTES.md`:

1. Which tool was hardest to scope without breaking its usefulness, and where did you draw the line?
2. Walk a concrete prompt-injection attempt (an instruction hidden in a ticket or customer message telling the agent to take an irreversible action) through your design, and show exactly where it is stopped.
3. Where did you feel pressure to over-hydrate "to be safe," and what convinced you the leaner budget was actually *more* accurate?

## Stretch goals

- Add an **audit/observability** design: what every privileged tool call records, and how the platform could revoke or narrow a tool centrally without touching every agent.
- Show how your hydration plan interacts with the GraphQL-vs-REST choice of [exercise-03](exercise-03-workflow-toolchain-integration.md): which layer is best served by a single shaped GraphQL read?
- Re-bind the entire design to the *other* scenario (SDLC ↔ support) and list what changed — proving the trust/hydration architecture is domain-independent ([Chapter 5](../05-beyond-sdlc-generalizing-the-platform.md)).
