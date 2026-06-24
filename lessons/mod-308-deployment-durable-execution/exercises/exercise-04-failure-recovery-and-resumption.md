# exercise-04: Failure Recovery and Resumption

**Estimated effort:** 3 hours

## Objective

Build the **failure-recovery design** for an agent fleet: a recovery matrix spanning worker, zone, and dependency failure; a fleet-scaling model that autoscales on the right signal; and isolation mechanisms (poison-run quarantine, per-run resource caps) so one bad run cannot take down the fleet. The deliverable is a recovery matrix, a scaling spec, and the isolation policies — an artifact that proves no in-flight run is lost across the failure scopes.

This is an **architecture** exercise. You are designing recovery behavior and capacity policy, not implementing an autoscaler.

## Background

This exercise covers material from:

- [Chapter 4 — Scaling and Failure Recovery for Agent Fleets](../04-fleet-scaling-and-recovery.md)
- [Chapter 1 — Durable Execution and Resumption](../01-durable-execution-and-resumption.md) — recovery is "reschedule the unfinished step," which only works because state is durable and steps are idempotent.

## The scenario

The fleet from exercises 01–03 in production:

> **50,000 runs/day.** ~90% finish in under a minute; ~8% run for hours; ~2% sleep for days on human approvals. Every active step calls an LLM API and/or an external tool with its own rate limit. Last month an availability-zone outage took out a third of the workers and **every agent errored out**, because run state lived in the workers. The team also hit an incident where a single malformed input crashed worker after worker as the engine kept rescheduling it.

Design the fleet so neither incident recurs.

## Tasks

### 1. Fix the scaling signal

- The team currently autoscales on **concurrent run count**. Explain why that is wrong for this workload (the 2% sleeping approvals dominate the count but consume ~zero compute), per [Chapter 4](../04-fleet-scaling-and-recovery.md).
- Specify the correct signal: **ready-step backlog** (or task schedule-to-start latency), **bounded by downstream rate limits**. Show the bound: `autoscale_target ≈ min(steps/sec demand, dependency_rate_limit / steps_per_call)`.
- Produce the three sizing artifacts: worker pool **floor/ceiling**, per-dependency **concurrency caps** (fleet backpressure), and a **tail budget** (max run age / reserved capacity for an approval-wave resume).

### 2. Build the recovery matrix

Fill in, per [Chapter 4](../04-fleet-scaling-and-recovery.md):

| Scope | Failure | Detection signal | Recovery action | Resulting guarantee |
| --- | --- | --- | --- | --- |
| Worker | pod OOM / crash / spot reclaim | | | |
| Zone / AZ | whole AZ goes dark | | | |
| Dependency | LLM/tool down or rate-limited | | | |

- For each scope, the recovery is **reschedule the unfinished step, not restart the run** — state *why* that holds (durable store HA, workers spread across zones with no affinity, idempotent steps, dependency failures pause rather than crash).

### 3. Fix the AZ-outage incident

- Trace exactly why the AZ outage caused total errors (run state lived in workers) and specify the three changes that turn an AZ loss into "surviving zones pick up orphaned runs": **multi-AZ durable store**, **workers spread across zones**, **no zone affinity on runs**.

### 4. Fix the poison-run incident

- Explain the failure mode: a deterministically-crashing run gets rescheduled, crashes the next worker, repeats — degrading the whole fleet.
- Specify a **per-run failure budget**: after N consecutive step failures, quarantine the run (dead-letter / manual-review state) instead of endless rescheduling. State N and the quarantine destination.

### 5. Bound blast radius

- Specify **per-run resource caps** (max concurrent activities, max total steps, max spend) so a runaway fan-out is bounded to a single run, per [Chapter 4](../04-fleet-scaling-and-recovery.md) (and connecting to the cost-incident theme of mod-307).

## Starter guidance

The recovery throughline to anchor on:

```text
   because progress is DURABLE:
     recovery = "reschedule the unfinished step"   (NOT "restart the run")
   true at every scope — worker, zone, dependency — IF:
     • durable store is multi-AZ HA           (the one thing you can't make stateless)
     • workers span zones, runs have no zone affinity
     • steps are idempotent (exercise-01)     (re-run ≠ double side effect)
     • dependency failure PAUSES the run durably (cheap wait), not crashes it
```

A scaling-signal sketch to make concrete:

```text
   run_count           ──▶ misleads (sleeping approvals inflate it)
   ready_steps_backlog ──▶ tracks real compute demand   ◀── autoscale on THIS
   dependency_rate_lim ──▶ caps useful worker count     ◀── bound by THIS

   target ≈ min( ready_steps/sec, rate_limit / steps_per_call )
```

An isolation-policy shape to fill in:

```text
   per-run failure budget : N consecutive step failures → QUARANTINE (dead-letter)
   per-run resource caps  : max_concurrent_activities, max_total_steps, max_spend
   → one bad run cannot crash workers fleet-wide or exhaust the pool
```

## Acceptance criteria

You can demonstrate that:

- You replace run-count autoscaling with **ready-step backlog bounded by dependency rate limits**, and produce floor/ceiling, per-dependency caps, and a tail budget.
- The recovery matrix is complete across worker/zone/dependency, and each row's recovery is "reschedule the step," with the four enabling conditions named.
- The AZ-outage fix names the multi-AZ store, cross-zone workers, and no-affinity changes, and traces how they convert an AZ loss into a non-event.
- The poison-run fix specifies a per-run failure budget with a quarantine destination.
- Per-run resource caps bound a runaway run's blast radius to itself.

## Reflection

In `NOTES.md`:

1. Walk a single run through a worker OOM that fires *after* it created an ERP record but *before* the step was recorded. Show how exercise-01's idempotency makes the rescheduled step safe.
2. Your durable store is the one component you cannot make stateless. What is your availability and backup design for it, and what is the blast radius if it goes down?
3. A batch of 5,000 approvals all gets clicked at 9am Monday, resuming 5,000 runs at once. How does your tail budget / reserved capacity prevent that wave from starving fresh runs?

## Stretch goals

- Design **graceful degradation** for a partial LLM-provider outage: a circuit breaker that fails over to a fallback model (mod-307 territory) vs. pausing runs durably until the primary returns. When do you pick each?
- Add a **chaos drill**: define an experiment that kills a random third of workers (and separately, a whole zone) and the success criterion (zero lost runs, bounded recovery time). What metric proves it passed?
- Reconcile the recovery matrix with the deploy runbook from [exercise-03](exercise-03-agent-fleet-deployment-strategy.md): is a deploy just a *planned* worker failure your recovery design already handles? Where does it differ?
