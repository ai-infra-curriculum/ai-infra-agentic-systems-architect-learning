# exercise-01: Durable Execution Design

**Estimated effort:** 3 hours

## Objective

Take a long-running, side-effecting agent and produce the **durable-execution design** for it: the split between the durable workflow and the replaceable activities, the replay/determinism boundary, and an idempotency strategy for every irreversible action. The deliverable is a design artifact an engineer could implement on Temporal (or an equivalent substrate) without having to make any of these decisions themselves.

This is an **architecture** exercise. You will not implement a working agent; you will produce diagrams, a decomposition table, and an idempotency register. Pseudocode is welcome where it clarifies a boundary; a running system is not the goal.

## Background

This exercise covers material from:

- [Chapter 1 — Durable Execution and Resumption](../01-durable-execution-and-resumption.md)

Assume your team can already build an agent loop and call tools (the AI-Engineer-track skill). Your job is the layer above: deciding what is durable, what is ephemeral, and how the run survives a process death.

## The scenario

Design durable execution for a **vendor-onboarding agent**:

> Given a new vendor, the agent: (1) collects three documents via a vendor portal (the vendor may take days to upload), (2) runs a sanctions-screening check and a credit check against two external APIs, (3) creates the vendor record in an ERP system and a payments record in a billing system, (4) waits for a compliance officer's sign-off, and (5) emails the vendor a welcome packet and marks onboarding complete.

The run routinely spans **several days**, takes **irreversible side effects** (account creation, the welcome email), and **waits on external events** (document upload, human sign-off). It is exactly the shape that mandates durable execution.

## Tasks

### 1. Draw the durable/ephemeral split

- Produce a diagram (ASCII or any tool) that places each part of the scenario into the **durable plane** (workflow state) or the **ephemeral plane** (activities/workers), per [Chapter 1](../01-durable-execution-and-resumption.md).
- The agent's control flow (the orchestration of steps 1–5) belongs in the durable plane; every LLM call, external API call, and side effect belongs in an activity. Show this explicitly.

### 2. Build the decomposition table

For each step in the scenario, fill a row:

| Step | Workflow or Activity? | Deterministic? | Why |
| --- | --- | --- | --- |

- Justify each placement. Anything non-deterministic (an LLM call, a `now()`, a random value, a network read) **must** be an activity whose result is recorded — explain where each one lives.
- Call out at least one place where a naive implementation would put non-determinism *inside* the workflow, and explain why that would corrupt replay.

### 3. Define the replay/determinism boundary

- Write the rule, in your own words, for what the **workflow code may not do** (read wall clock, generate randomness, hit the network, branch on un-recorded values) and how each of those needs is satisfied instead (durable timer, activity, recorded result).
- Pick one concrete step (e.g., "wait until the vendor uploads documents") and show how it is expressed durably (a signal/timer the workflow sleeps on) rather than a polling loop in process memory.

### 4. Build the idempotency register

- Enumerate **every irreversible side effect** in the scenario (account creations, the email, any external write).
- For each, specify an **idempotency key** derived from stable run/step identity (not wall-clock time, not a fresh UUID) and the dedup strategy (downstream API accepts the key, or you record-and-replay the result locally), per [Chapter 1](../01-durable-execution-and-resumption.md).

| Side effect | Idempotency key | Dedup strategy | What "ran twice" must equal |
| --- | --- | --- | --- |

### 5. Justify the substrate

- Using the substrate table from [Chapter 1](../01-durable-execution-and-resumption.md), argue which substrate (roll-your-own checkpoints, durable functions, a workflow engine, a managed step orchestrator) you would choose for this scenario, and why the cheaper options are insufficient here.

## Starter guidance

A decomposition sketch to react to (refine it; do not adopt it uncritically):

```text
   WORKFLOW (durable, deterministic):
     orchestrate steps 1→5
     sleep on signal: "documents uploaded"
     sleep on signal: "compliance sign-off"   (see exercise-02)
     decide branch on RECORDED activity results only

   ACTIVITIES (ephemeral, may run >once → must be idempotent):
     A1 request_documents(vendor)        ← side effect: portal task
     A2 sanctions_check(vendor)          ← external API read
     A3 credit_check(vendor)             ← external API read
     A4 create_erp_record(vendor)        ← IRREVERSIBLE write  * idempotency key
     A5 create_billing_record(vendor)    ← IRREVERSIBLE write  * idempotency key
     A6 send_welcome_email(vendor)       ← IRREVERSIBLE send   * idempotency key
```

An idempotency-key shape to adapt:

```text
   key = stable_hash(run_id, step_id, payload_digest)
   on activity call:
     if store.has(key): return store.get(key)     # replayed — return recorded result
     result = do_side_effect()
     store.put(key, result)                       # record BEFORE returning
     return result
```

A sequence diagram to complete — show where a crash is survivable and where a double side effect would occur without an idempotency key:

```text
   workflow        activity (A4: create_erp_record)        ERP API
   ────────        ────────────────────────────────        ───────
      │  dispatch A4  │                                        │
      │──────────────▶│   create record ──────────────────────▶│  (record created)
      │               │   ◀──────────────── 200 + record_id ───│
      │               │  ✗ worker dies BEFORE reporting result │
      │  (engine sees no completion → reschedules A4)           │
      │  dispatch A4  │                                        │
      │──────────────▶│   create record ──────────────────────▶│  (?? duplicate ??)
      │               │   ── your idempotency design must make  │
      │               │      this return the SAME record_id ────│
```

You do **not** need the deployment strategy (exercise-03) or the full HITL contract (exercise-02) here — note where step 4 hands off to a HITL gate, but design that gate in exercise-02.

## Acceptance criteria

You can demonstrate that:

- Every step is placed in the durable or ephemeral plane with a justification, and **all non-determinism lives in activities**, not the workflow.
- The decomposition table is complete and at least one "naive non-determinism in the workflow" trap is identified and explained.
- The replay/determinism rule is stated correctly, and at least one wait is expressed as a durable signal/timer rather than an in-memory loop.
- The idempotency register covers **every** irreversible side effect, each with a stable key and a dedup strategy, such that re-running a step is provably safe.
- The substrate choice is argued against the alternatives, not asserted.

## Reflection

In `NOTES.md`:

1. Which step was hardest to classify as workflow-vs-activity, and what tipped your decision?
2. Trace the crash sequence in the starter diagram (worker dies *after* the ERP write but *before* it is recorded) and show how your idempotency design prevents a duplicate ERP record.
3. Where would deterministic replay break if a future engineer "optimized" the workflow by adding a `time.time()` call inside it? What would they see?

## Stretch goals

- Add a **compensation** (saga) design: if the credit check fails *after* the ERP record is created, what undoes the ERP record? Sketch the compensating activities and where they fire.
- Show how a **workflow-version change** (you add a fourth check) would interact with a run already in flight — and connect it to the rainbow-deploy / patching decision in [exercise-03](exercise-03-agent-fleet-deployment-strategy.md).
- Estimate the durable-store write volume (events per run) and argue whether the auditability payoff justifies it for this workload.
