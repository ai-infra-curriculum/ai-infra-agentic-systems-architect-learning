# exercise-02: Handoffs vs. Manager-as-Tools

**Estimated effort:** 3 hours

## Objective

For a multi-specialist customer-facing system, choose a delegation model — **handoffs** or **manager-as-tools** — and assign **answer ownership** explicitly. You'll produce a delegation diagram, an ownership decision, and the transfer/return contracts. The point is to make the "who speaks to the user and who is accountable" decision deliberately, not by accident.

## Background

This exercise covers material from:

- [Chapter 2 — Handoffs vs. Manager-as-Tools, and Answer Ownership](../02-handoffs-vs-manager-as-tools.md)

Stay at the design altitude: you are specifying control flow and contracts, not implementing agents.

## Prerequisites

- Chapter 2 read.
- Chapter 1 helpful for the context-cost comparison.

## System

> *A support assistant for a SaaS product with three specialist domains: **billing** (refunds, plan changes), **technical troubleshooting** (multi-turn diagnosis), and **account/security** (password resets, access — a risk-sensitive domain). A user request can touch one or more domains in a single conversation.*

## Tasks

### 1. Pick the delegation model

- Choose handoffs, manager-as-tools, or a justified blend. Tie the choice to the system's needs: consistent voice, specialist autonomy, and where guardrails must live.

### 2. Assign answer ownership

- For each domain, state **who owns the final user-facing answer** and **who the user hears**. There must be exactly one owner at every point in the conversation — no orphaned ownership.

### 3. Draw the control flow

- ASCII diagram showing delegation edges. For handoffs, draw the **handback** edges explicitly and name the new owner on each transfer. For manager-as-tools, show specialists returning to the manager.

### 4. Specify the contracts

- For each transfer (handoff) or call (manager-as-tools), define the typed payload: handoffs carry `{reason, user_intent, state, new_owner}`; manager-as-tools calls carry `{request, return_shape}`.

### 5. Guardrail placement

- Account/security is risk-sensitive. State where the safety/authorization guardrail lives and prove it can't be bypassed by a transfer. (If you chose handoffs, this is the hard part.)

## Starter guidance

Use this decision matrix and the two diagram templates; fill the one you choose.

| Decision | Choice | Why |
| --- | --- | --- |
| Delegation model | | |
| Answer owner (billing) | | |
| Answer owner (technical) | | |
| Answer owner (security) | | |
| Guardrail choke point | | |
| Transfer/return contract | | |

```text
HANDOFF MODEL (control transfers; receiver owns the answer)

  user ──▶ triage ──handoff{reason,intent,state,owner}──▶ billing ──▶ user
                │                                            │
                │◀───────────── handback{result,owner} ──────┘
                └─ names new owner on every edge
```

```text
MANAGER-AS-TOOLS MODEL (manager always owns the answer)

  user ──▶ manager ──┬─call{request}──▶ billing   ──return{result}──┐
                     ├─call{request}──▶ technical ──return{result}──┤
                     └─call{request}──▶ security  ──return{result}──┘
              manager composes ──▶ user
```

No agent code is required. The deliverable is the diagram + matrix + contracts.

## Acceptance criteria

You can demonstrate that:

- The delegation model is chosen and justified against voice, autonomy, and guardrail placement.
- Every domain has exactly one named answer owner; no point in any conversation has zero or two owners.
- The diagram shows handback edges (handoff) or return edges (manager-as-tools), with the owner named on each transfer.
- Transfer/return contracts are typed and carry enough to avoid context loss.
- The security guardrail's choke point is identified and shown to be un-bypassable by transfer.

## Reflection

In `NOTES.md`:

1. Trace a request that touches all three domains. Where does ownership move, and is the user's voice consistent or shifting? Is that the right call for *this* product?
2. Estimate the context-cost difference between your model and the alternative for that three-domain request.
3. Construct the handoff-loop (ping-pong) failure for your design and state the bound that prevents it.

## Stretch goals

- Design the *blend*: a manager owns the answer but hands off to the technical specialist for a long diagnosis, then takes ownership back. Draw both transitions and name the owner at each step.
- Add a fourth specialist and show whether your model scales or starts to strain (voice drift for handoffs; manager context bloat for manager-as-tools).
- Write the one-paragraph accountability statement: when a wrong answer ships, whose design owns the failure under your model?
