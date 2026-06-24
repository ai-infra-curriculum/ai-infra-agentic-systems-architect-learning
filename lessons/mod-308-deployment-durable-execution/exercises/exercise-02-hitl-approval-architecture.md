# exercise-02: HITL Approval Architecture

**Estimated effort:** 3 hours

## Objective

Design a **human-in-the-loop approval gate** as a durable, resumable interrupt: one that survives an arbitrary wait and a process restart, supports the right subset of the approve/edit/reject/respond decision set, and carries a complete authority/timeout/audit contract. The deliverable is a state-machine spec and a sequence diagram an engineer could implement on a durable substrate (Temporal signals, LangGraph `interrupt`, or equivalent).

This is an **architecture** exercise. You are specifying the gate's contract and lifecycle, not building an approval UI.

## Background

This exercise covers material from:

- [Chapter 3 — Human-in-the-Loop Approval Architecture](../03-human-in-the-loop-architecture.md)
- [Chapter 1 — Durable Execution and Resumption](../01-durable-execution-and-resumption.md) — the gate is built on durable interrupts and idempotent resume.

It continues the vendor-onboarding scenario from [exercise-01](exercise-01-durable-execution-design.md): step 4 is "wait for a compliance officer's sign-off." Here you design that gate properly, and a second, higher-stakes gate.

## The scenario

Two gates to design:

1. **Compliance sign-off** (from exercise-01): before the vendor goes live, a compliance officer must review the screening results and sign off. The officer may approve, may **edit** the vendor's risk tier before approving, may **reject** (sending the agent back to request more documents), or may **respond** with a question for the agent to address.
2. **Wire transfer** (new): a downstream finance agent can initiate a vendor payment. Transfers over **$50,000** require **two** approvers from the finance role. This gate allows approve/edit/reject but **not** respond.

## Tasks

### 1. Define the decision set per gate

- For each gate, specify exactly which of **approve / edit / reject / respond** are valid, and what each does to the run's state (resume past the gate, resume with edited args, return control to the agent to replan, inject a response as an observation), per [Chapter 3](../03-human-in-the-loop-architecture.md).
- Justify any decision you *exclude* (e.g., why "respond" is invalid for the wire-transfer gate).

### 2. Draw the durable-interrupt lifecycle

- Produce a sequence diagram showing: the agent reaching the gate, persisting `{gate, proposed action, allowed decisions, deadline}`, **yielding its worker**, the process being free to die, the human deciding minutes-to-days later, and the run resuming on **any** worker.
- Mark the point where the process can be killed/deployed/drained and explain why the run survives it (state is in the durable plane, not the process).

### 3. Write the approval contract

For each gate, fill in the contract from [Chapter 3](../03-human-in-the-loop-architecture.md):

| Field | Compliance sign-off | Wire transfer |
| --- | --- | --- |
| allowed_decisions | | |
| authority_policy (who, thresholds, multi-approver) | | |
| timeout (deadline + default action) | | |
| correlation_key | | |
| audit_record (fields captured) | | |
| idempotency (duplicate decision handling) | | |

- The wire-transfer row must encode the **$50k / two-approver** rule and validate the **decider's identity** on the signal, not trust it from the UI.

### 4. Specify idempotent resume

- Show how a decision delivered **twice** (the approval API retried) resumes the run exactly once. Use the at-least-once idempotency discipline from [Chapter 1](../01-durable-execution-and-resumption.md): the gate consumes the first valid decision for its `(run_id, gate_id)` and ignores duplicates.

### 5. Connect to deploy and tail policy

- Per [Chapter 2](../02-inflight-safe-deployment.md) and [Chapter 4](../04-fleet-scaling-and-recovery.md): an approval that can wait forever pins an old code version alive and stretches the fleet's tail. State your **approval TTL** and what happens at the deadline for each gate, and explain how that TTL also bounds version sprawl.

## Starter guidance

A gate state machine to complete:

```text
   ┌─────────┐  reach gate   ┌──────────┐  valid decision   ┌──────────┐
   │ RUNNING │──────────────▶│ WAITING  │──────────────────▶│ RESUMED  │
   └─────────┘  persist +    └────┬─────┘   (approve/edit/   └──────────┘
                yield worker      │          reject/respond)
                                  │  deadline passes
                                  ▼
                            ┌──────────────┐
                            │ TIMED_OUT    │── default action:
                            └──────────────┘   reject | escalate | notify
```

A decision payload shape to adapt:

```text
   decision = {
     gate_id,                 # correlates to the waiting run; stable across the wait
     run_id,
     decision,                # approve | edit | reject | respond
     edited_args?,            # present only for edit
     response_text?,          # present only for respond
     decider_identity,        # VALIDATED against authority_policy, not trusted
     decided_at,
   }
   # idempotency: first valid decision for (run_id, gate_id) wins; duplicates ignored.
```

The authority sketch for the wire-transfer gate (make it concrete):

```text
   amount <= $50k  → 1 approver, role=finance
   amount >  $50k  → 2 distinct approvers, role=finance
   default on timeout (24h) → REJECT (irreversible action; safe default)
```

You do **not** need fleet scaling math (exercise-04) here — note where the TTL connects, but size capacity there.

## Acceptance criteria

You can demonstrate that:

- Each gate has an explicit, justified decision set, and the four actions are correctly distinguished by *what they do to the run's state*.
- The durable-interrupt lifecycle shows the run surviving a process death during the wait, resuming on any worker via a signal.
- Both approval contracts are complete; the wire-transfer contract correctly encodes the $50k two-approver rule and validates decider identity server-side.
- A duplicate decision provably resumes the run exactly once.
- Each gate has a TTL with a default action, and you explain its effect on rainbow-deploy version sprawl and fleet tail.

## Reflection

In `NOTES.md`:

1. Why is a blocking `input()`-style approval wrong for these gates? Trace exactly what breaks when a deploy rolls the pod mid-wait.
2. The "edit" action lets a human change the agent's proposed arguments. What new risk does that introduce, and how does your audit record make it accountable?
3. For the wire-transfer gate, what stops a single malicious approver from self-approving twice to satisfy the two-approver rule?

## Stretch goals

- Add an **escalation ladder**: if the first approver does not respond in 4h, notify a second; at 24h, escalate to a manager. Express it as durable timers, not a polling loop.
- Design the **proposal payload** the gate surfaces to the human (action, arguments, agent rationale, screening evidence) so the decision is informed rather than a rubber stamp.
- Reconcile this gate-placement with the idempotency register from [exercise-01](exercise-01-durable-execution-design.md): show that the gated actions and the irreversible side effects are (nearly) the same set, and explain any divergence.
