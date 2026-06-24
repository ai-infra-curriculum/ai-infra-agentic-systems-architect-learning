# exercise-01: Orchestrator-Worker Topology Design

**Estimated effort:** 3 hours

## Objective

Design — on paper, not in code — an orchestrator-worker topology for a real, broad workload, and **defend its token budget**. You'll produce a topology diagram, a worker assignment table, and a token-economics estimate that shows the ~15x multiplier is justified (or shows it isn't, and a single agent wins). The deliverable is an architecture artifact a team could build from.

## Background

This exercise covers material from:

- [Chapter 1 — Orchestrator-Worker Topologies and Token Economics](../01-orchestrator-worker-topologies.md)

You are the architect. You are *not* writing the agent loop — you are deciding its shape and proving the budget. Assume a competent engineer implements whatever you specify.

## Prerequisites

- Chapter 1 read.
- A workload to design for. Use the provided one or bring your own (it must decompose into independent breadth, or there is no fan-out to design).

## Workload

> *"Build a competitive-intelligence brief on the five largest managed-Postgres providers, covering pricing, autoscaling, backup/restore guarantees, region coverage, and a recent-incident summary for each."*

## Tasks

### 1. Decide multi-agent vs. single agent

- State in two or three sentences whether this workload earns a multi-agent system at all. Name the specific property (independent breadth, context overflow, specialization) that justifies — or fails to justify — fan-out.

### 2. Draw the topology

- Produce an ASCII topology diagram (decompose → fan-out → synthesize). Label N (the fan-out cap), the worker roles, and where distilled returns flow back.
- If you choose a hierarchical topology, justify the extra layer against its compounded cost.

### 3. Worker assignment table

- Fill a table of `{worker_role, instruction summary, tools needed, expected return shape}`. Each assignment must be **self-contained** (a worker can't see the conversation) and **non-overlapping** with the others.

### 4. Token-economics estimate

- Use the envelope from Chapter 1 to produce a rough estimate. You don't need exact token counts — you need defensible relative magnitudes and the ratio that matters.
- Show the **distilled-return ratio**: estimate orchestrator context growth *with* distilled returns vs. if workers returned full transcripts.
- State the resulting multiplier band (e.g., "~12–15x a single chat") and whether the workload's value justifies it.

### 5. Bounds and degradation

- State the fan-out cap, the per-worker turn cap, and what the system returns if 2 of 5 workers fail.

## Starter guidance

Use this topology diagram template, token envelope, and decision matrix as your skeleton.

```text
TOPOLOGY: <name>            FAN-OUT CAP N = <n>

                ┌──────────────┐
   task ───────▶│ orchestrator │  decompose
                └──────┬───────┘
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌─────────┐    ┌─────────┐    ┌─────────┐
   │ role A  │    │ role B  │    │ role C  │   distilled return each
   └────┬────┘    └────┬────┘    └────┬────┘
        └──────────────┼──────────────┘
                       ▼
                ┌──────────────┐
                │ orchestrator │  synthesize ──▶ brief
                └──────────────┘
```

```text
TOKEN ENVELOPE

est ≈ orchestrator_overhead
    + N × (shared_context + worker_work + distilled_return)
    + synthesis_pass

distilled_return / worker_work       = ____   (target ≪ 1)
estimated multiplier vs single chat  = ____x
verdict: [ justified | not justified ] because ____
```

| Decision | Choice | Why |
| --- | --- | --- |
| Multi-agent? | | |
| Topology shape | | |
| Fan-out cap N | | |
| Return distillation | | |
| Failure behavior | | |

You do **not** need to write or run any agent code for this exercise.

## Acceptance criteria

You can demonstrate that:

- The multi-agent-vs-single decision is stated and tied to a specific property of the workload.
- The topology diagram labels N, worker roles, and distilled-return flow.
- The assignment table's rows are self-contained and non-overlapping.
- The token estimate shows the distilled-return ratio and a defended multiplier band.
- Fan-out cap, turn cap, and 2-of-5-failure behavior are all specified.

## Reflection

In `NOTES.md`:

1. At what workload size would a *single agent* have been the better call here, and why?
2. If your distilled returns were instead full transcripts, estimate how much the orchestrator's context (and cost) grows. What does that do to the multiplier?
3. Where, if anywhere, would hierarchy (sub-orchestrators) pay for its compounded cost on this workload?

## Stretch goals

- Re-budget the same workload at N = 10 and N = 3. Plot the cost/coverage tradeoff and pick a defensible N.
- Add a bounded *second round*: after synthesis, the orchestrator spawns follow-up workers to fill gaps. Show the added cost and the cap that keeps it from running away.
- Redesign as a sequential chain instead of a fan-out and argue which is right for *this* workload.
