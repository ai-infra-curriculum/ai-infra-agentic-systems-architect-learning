# exercise-04: Coordination and Escalation Design

**Estimated effort:** 4 hours

## Objective

Write the **coordination and escalation specification** for a multi-agent system: the loop bounds, the deadlock rules, the escalation ladder, and the human gates. This is the half of the design that governs what happens when the happy path breaks. The deliverable is a one-page spec an engineer implements verbatim.

## Background

This exercise covers material from:

- [Chapter 4 — Coordination and Escalation Strategy](../04-coordination-and-escalation.md)

You specify the rules; you do not write the loop. Every rule must be concrete enough to implement without asking you a follow-up.

## Prerequisites

- Chapter 4 read.
- Chapters 1–3 helpful — coordination interacts with topology, ownership, and protocols.

## System

> *An orchestrator-worker travel-planning assistant. The orchestrator fans out to workers for flights, lodging, and local activities; workers can disagree (e.g., a flight worker and a lodging worker pick incompatible dates), and some bookings are irreversible and over a cost threshold. The assistant must never invent a booking that doesn't exist.*

## Tasks

### 1. Coordination mechanism

- Choose how the workers stay consistent: orchestrator-mediated, shared blackboard, or explicit message contracts. Justify the *minimum* mechanism that keeps the system coherent, and name the concurrency surface you've taken on if you pick a blackboard.

### 2. Conflict-resolution rule

- Workers can return contradictory plans (incompatible dates). Write the deterministic tie-breaker and what happens when it can't decide.

### 3. Loop bounds

- Specify hard caps: turns per worker, total worker count / fan-out depth, evaluator or re-decompose iterations, total wall-clock, and total spend. State what a tripped bound *does* (it steps up the ladder; it does not crash).

### 4. Escalation ladder

- Write the rungs from cheapest recovery to safe failure (retry → reassign/re-decompose → degrade → human → safe-failure). Map at least one concrete failure from this system onto each rung.

### 5. Human gates

- Place the human gate(s). For each: trigger, what the human sees, the timeout default, and who is accountable. The irreversible over-threshold booking must be gated.

### 6. Partial failure behavior

- Specify exactly what the assistant returns if the activities worker fails but flights and lodging succeed. It must flag the gap honestly, not paper over it.

## Starter guidance

Fill this escalation-ladder template and the bounds/gates tables.

```text
ESCALATION LADDER

  rung 0  retry once          ── trigger: ____   ── this system: ____
  rung 1  reassign/re-decompose── trigger: ____   ── this system: ____
  rung 2  degrade (partial)   ── trigger: ____   ── this system: ____
  rung 3  human-in-the-loop    ── trigger: ____   ── this system: ____
  rung 4  safe failure         ── trigger: ____   ── this system: ____

  invariant: the model may NEVER fabricate to avoid a higher rung.
```

| Loop bound | Cap | On trip |
| --- | --- | --- |
| Turns per worker | | |
| Fan-out depth / agent count | | |
| Re-decompose iterations | | |
| Total wall-clock | | |
| Total spend | | |

| Human gate | Trigger | Human sees | Timeout default | Accountable |
| --- | --- | --- | --- | --- |
| Irreversible booking > threshold | | | | |
| Unresolvable worker conflict | | | | |

No agent code is required. The deliverable is the one-page spec.

## Acceptance criteria

You can demonstrate that:

- A minimum coordination mechanism is chosen and justified; any added concurrency surface is named.
- The conflict-resolution rule is deterministic and has a defined "can't decide" path.
- Every loop bound has a numeric cap and a "what a trip does" entry that steps up the ladder rather than crashing.
- The escalation ladder has all five rungs, each mapped to a concrete failure from this system, with the no-fabrication invariant stated.
- Every human gate specifies trigger, what the human sees, a timeout default, and accountability — and the irreversible over-threshold booking is gated.
- Partial-failure behavior flags the gap honestly.

## Reflection

In `NOTES.md`:

1. Which loop bound did you find hardest to set a number for, and how would you tune it from production data?
2. Construct the deadlock (worker conflict that ping-pongs, or a gate with no timeout default) your spec prevents. Show the exact rule that breaks it.
3. Where is the system most tempted to fabricate rather than escalate, and what in your spec makes the honest path the easy one?

## Stretch goals

- Add a shared blackboard so workers can see each other's date picks mid-flight and avoid the conflict up front. Specify the locking/consistency rule and weigh it against the simpler orchestrator-mediated design.
- Extend the human gate into a *review* gate that samples even successful plans, and state the sampling rate and what reviewers feed back into the system.
- Combine with exercise-01's topology: show how your bounds and ladder change if the fan-out becomes hierarchical (sub-orchestrators), and which caps must move to the system level.
