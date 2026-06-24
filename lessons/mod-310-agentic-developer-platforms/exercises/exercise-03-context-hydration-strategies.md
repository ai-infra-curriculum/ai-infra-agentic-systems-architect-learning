# exercise-03: Context-Hydration Strategies for SDLC Tasks

**Estimated effort:** 3 hours

## Objective

Design a layered, just-in-time context-hydration strategy for a real SDLC task and defend its context budget. You will assign each piece of context to a layer (standing / task / retrieved-on-demand / distilled sub-context), specify how each layer is delivered (skill, event payload, MCP tool, subagent), estimate the token budget, and design the safe-degradation behavior when context is missing. The deliverable is a hydration plan that is small, fast, and *more* accurate than a context dump.

## Background

This exercise covers material from:

- [Chapter 3 — Reliable Tool-Use Patterns and Context Hydration](../03-tool-use-and-context-hydration.md)

The discipline: **pull, don't pre-load; distill, don't dump.** A maximal context is not a safe default — it is slower, costlier, and less accurate.

## Prerequisites

- Read Chapter 3.
- Pick one SDLC task to design for: "debug a failing CI test," "implement a small ticket," or "triage a production incident from logs."
- Rough token intuition (an order-of-magnitude estimate per source is enough).

## Tasks

### 1. Inventory the candidate context

- List everything the agent *could* be given for your task: the goal, conventions, code files, tickets, logs, design docs, history. Do not filter yet — inventory first.

### 2. Assign each item to a layer

- Place each item in L0 (standing), L1 (task), L2 (retrieved-on-demand), or L3 (isolated/distilled), and name the delivery mechanism for each (plugin skill, event payload, MCP tool call, subagent).
- Justify why the bulk of the context lives in L2 (lazy pull), not L0/L1 (always present).

### 3. Budget the context

- Estimate the token cost of each layer for one run. Contrast it with a naive "load the whole repo + all tickets" baseline, and quantify the gap.

### 4. Identify what to *exclude*

- List the context you deliberately leave out (the rest of the repo, unrelated tickets, full history) and explain why each exclusion makes the agent *more* accurate, not less.

### 5. Design safe degradation

- For two realistic "missing context" cases (a one-line ticket with no acceptance criteria; an MCP fetch that errors), specify the agent's correct behavior: make sparseness legible, ask vs. assume, surface the failure — never hallucinate around it.

### 6. Write the plan

- Produce the final table: `layer | content | source/mechanism | budget`, plus the degradation rules.

## Starter guidance

Target shape (for "debug a failing CI test"):

```text
| Layer | Content                                   | Source / mechanism | Budget    |
| ----- | ----------------------------------------- | ------------------ | --------- |
| L0    | debug conventions; "don't disable tests"  | plugin skill       | ~tiny     |
| L1    | failing test name, assertion, CI log tail | CI event payload   | ~small    |
| L2    | test file + source under test + last 3    | MCP, on demand     | ~bounded  |
|       | commits that touched them                 | (search_code)      |           |
| L3    | cross-module failures → subagent traces   | subagent           | ~distilled|
|       | call path, returns 5-line root cause      |                    |  (summary)|
```

Degradation example:

```text
If get_ticket() returns no acceptance criteria:
  → do NOT invent requirements
  → state "ticket is underspecified: <what's missing>"
  → ask one clarifying question OR escalate to the human
```

## Acceptance criteria

You can demonstrate that:

- Every context item is assigned to a layer with a named delivery mechanism, and the bulk lives in L2 (pulled on demand), not pre-loaded.
- The budget table shows a clear token (and latency) advantage over a load-everything baseline, with numbers.
- The exclusion list explains how leaving context out *improves* accuracy via reduced noise.
- The degradation rules make missing context legible and prefer asking/surfacing over hallucinating, for both cases.

## Reflection

In `NOTES.md`:

1. Which item was tempting to put in L0/L1 "to be safe" but belonged in L2? What did moving it lazy buy you?
2. Estimate how much accuracy you'd *lose* by dumping the whole repo into context. Why does more context hurt here?
3. When would an L3 subagent's distillation actually lose information the main agent needed, and how would you detect that?

## Stretch goals

- Add a caching layer: which L2 reads are stable enough to cache across runs, and what's the invalidation trigger?
- Design a "context budget guardrail": a check that warns or blocks when a single run's hydration exceeds a threshold, and where it lives (hook vs. tool-side).
- Reconcile this with Chapter 2: show that your L2 retrieval respects the requesting human's permissions, so lazy hydration doesn't become a data-leak path.
</content>
