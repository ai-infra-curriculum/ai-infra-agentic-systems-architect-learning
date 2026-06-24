# Chapter 1 — Durable Execution and Resumption

A short agent run is stateless from the operator's point of view: if it crashes, you retry the whole thing. That model collapses the moment a run lasts longer than the lifetime of a single process. A research agent reading 200 documents, a procurement agent waiting two days on a vendor, an incident agent paused on a human approval — each of these will, with near-certainty, outlive the process it started in. A deploy will roll its pod, a node will drain, a spot instance will be reclaimed, an OOM killer will fire. **If the agent's progress lives only in process memory, every one of those events destroys the run.**

Durable execution is the architectural answer: make the run's *progress* a first-class, persisted thing, separate from the *compute* that advances it. The compute is ephemeral and replaceable; the progress is durable and resumable. This chapter is about that separation, the guarantees it buys, and the constraints it imposes on how you write the agent.

## The core split: workflow state vs. worker compute

The single most important idea in this module is one boundary:

```text
   ┌──────────────────────────────────────────────────────────┐
   │  DURABLE PLANE  (survives any process death)              │
   │                                                           │
   │   workflow state:  step history, inputs, results so far,  │
   │                    pending timers, signal/approval state  │
   │   ── persisted to a durable store (event log / DB) ──     │
   └───────────────────────────┬──────────────────────────────┘
                               │  dispatch next step / replay
                               ▼
   ┌──────────────────────────────────────────────────────────┐
   │  EPHEMERAL PLANE  (any process can be killed anytime)     │
   │                                                           │
   │   workers / activities:  the LLM call, the tool call,     │
   │                          the retrieval, the side effect   │
   │   ── stateless; hold nothing that must survive a crash ── │
   └──────────────────────────────────────────────────────────┘
```

The **durable plane** owns *what has happened and what happens next*. The **ephemeral plane** owns *doing the next thing*. A worker can die at any instant; the durable plane notices, and reschedules the unfinished step onto a fresh worker. The run does not restart — it resumes from the last durably recorded point.

Temporal is the reference implementation of this pattern (Cadence, AWS Step Functions, DBOS, Inngest, and Restate are siblings with different trade-offs). In Temporal's vocabulary:

- A **Workflow** is the durable orchestration — its progress is persisted as an **event history**.
- **Activities** are the ephemeral side-effecting steps (your LLM call, tool call, DB write).
- A **Worker** is a process that executes Workflow and Activity code; it holds no durable state and is freely replaceable.

The architect's deliverable is the *decomposition*: which logic is durable orchestration (the Workflow) and which is a replaceable side effect (an Activity). Get that boundary right and resumption is nearly free; get it wrong and you either lose state or re-run expensive side effects.

## How resumption actually works: deterministic replay

Durable engines like Temporal do not snapshot your program's memory. They **replay** it. When a Workflow needs to resume — after a crash, a deploy, or simply being evicted from a worker's cache — the engine re-executes the Workflow function from the top, feeding it the recorded **event history**. Each step that already completed returns its recorded result instead of running again; execution fast-forwards to the first not-yet-completed step and continues live from there.

```text
  Workflow function (re-executed on resume)
  ────────────────────────────────────────────
  step A  ──▶ history has result  ──▶ return recorded result (no re-run)
  step B  ──▶ history has result  ──▶ return recorded result (no re-run)
  step C  ──▶ NOT in history      ──▶ execute live, append result to history
  step D  ──▶ not reached yet     ──▶ runs after C
```

This replay model is what makes resumption possible without snapshotting memory — but it imposes a hard rule:

> **Workflow code must be deterministic.** Re-executing it with the same history must produce the same sequence of steps.

That means inside the durable orchestration you must **not** read the wall clock, generate a random number, hit the network, or branch on anything non-recorded — because on replay those would diverge from the recorded history and corrupt the run. All non-determinism is pushed into **Activities**, whose *results* are recorded. Need the current time? It comes from a durable timer, not `now()`. Need randomness or an LLM call? It happens in an Activity, and the recorded result is what replay sees.

For an agent, this maps cleanly: the **agent loop's control flow is the deterministic Workflow**; every **LLM call, tool call, and retrieval is an Activity**. The model's output is non-deterministic — which is exactly why it belongs in an Activity, where its result is recorded once and replayed thereafter.

## Idempotency: the price of at-least-once

Durable engines guarantee *at-least-once* execution of a step, not exactly-once. A worker can complete an Activity — charge a card, send an email, file a ticket — and die *before* it reports completion to the durable plane. The engine, seeing no completion, reschedules the Activity. Now the side effect runs twice.

This is not a Temporal quirk; it is a fundamental property of any system that recovers from crashes. The architect's job is to make every externally-visible side effect **idempotent** so that "ran twice" equals "ran once":

```text
  Activity:  "create vendor PO"
  ─────────────────────────────────────────────
  idempotency key = hash(run_id, step_id, payload)

  on call:
    if store.has(key):  return store.get(key)   # already done — replay result
    else:               result = do_side_effect()
                        store.put(key, result)   # record before returning
                        return result
```

The pattern: derive a **stable idempotency key** from the run and step identity (never from wall-clock time or a fresh UUID), and either (a) make the downstream API accept that key and dedupe server-side, or (b) record the result in a store keyed by it so a re-run returns the recorded result instead of re-doing the work. Every irreversible action an agent can take — payments, emails, ticket creation, external writes — needs one of these. As an architect, *enumerating the irreversible side effects and assigning each an idempotency strategy* is a required design artifact, not an implementation detail.

## What "durable" buys you — and what it costs

Durable execution gives you four properties that long-running agents cannot get any other way:

- **Crash resumption.** A killed worker loses no progress; the run continues on a fresh worker from the last recorded step.
- **Deploy survival.** Because progress lives in the durable plane, you can replace the workers entirely — this is the foundation for the in-flight-safe deploys of [Chapter 2](02-inflight-safe-deployment.md).
- **Arbitrary waits.** A Workflow can sleep for days waiting on a timer or a human signal without holding a process open — the foundation for the HITL gates of [Chapter 3](03-human-in-the-loop-architecture.md).
- **Auditability.** The event history *is* a complete, replayable record of what the agent did and why — invaluable for debugging non-reproducible agent trajectories and for compliance.

It is not free, and an architect must price the costs honestly:

- **Determinism is a real constraint** on how the orchestration is written, and a common source of subtle bugs (the dreaded "non-deterministic error" on replay).
- **Versioning the Workflow code is hard.** A run started under v1 may replay weeks later, after you have deployed v2. If v2 changed the step sequence, replay breaks. This is the central problem of [Chapter 2](02-inflight-safe-deployment.md).
- **Operational surface grows.** You now run and scale a durable engine and its persistence store, plus the worker fleet ([Chapter 4](04-fleet-scaling-and-recovery.md)).
- **Overkill for short runs.** A sub-minute, side-effect-free agent does not need any of this. Reserve durable execution for runs that genuinely cross process lifetimes or take irreversible actions.

## Choosing the substrate

You do not have to adopt a full workflow engine on day one. The substrates form a spectrum:

| Substrate | Durability model | Best when |
| --- | --- | --- |
| Roll-your-own (DB checkpoints) | You checkpoint state after each step; you write the resume logic | Few steps, simple state, you want no new infra |
| Durable functions (DBOS, Inngest, Restate) | Library/runtime persists step results; lighter than a full engine | You want durability inside your existing service |
| Workflow engine (Temporal, Cadence) | Full event-history replay, timers, signals, versioning | Long, multi-step, signal-driven agents; you need the strongest guarantees |
| Managed step orchestrators (AWS Step Functions) | State machine persisted by the cloud provider | You want a managed plane and can express the agent as a state machine |

The architectural decision is driven by three questions: *How long do runs live?* *Do they take irreversible side effects?* *Do they wait on external signals (human or vendor)?* The more "yes," the further down the table you belong. A run that is short, reversible, and never waits does not need durable execution at all — and forcing it in is the over-engineering failure mode this module warns against.

## Worked judgment

*"Summarize this 300-page contract."* — A single long LLM call, no side effects, completes in minutes. If it crashes, retry the whole thing. **No durable execution needed.** Checkpoint at most.

*"Onboard a new vendor: collect documents, run three compliance checks, create accounts in four systems, and wait for a compliance officer's sign-off before going live."* — Multi-day, irreversible side effects (account creation), a human gate. **Durable execution is mandatory.** Decompose into a Workflow (the orchestration + the human signal wait) and idempotent Activities (each account creation, each check), with idempotency keys on every external write.

## Key takeaways

- A long-running agent is a distributed system; its **progress must outlive the process** that produced it. Separate the **durable plane** (workflow state) from the **ephemeral plane** (workers/activities).
- Durable engines **resume by deterministic replay**, not memory snapshots — so orchestration code must be deterministic, and all non-determinism (LLM calls, tool calls, time, randomness) is pushed into recorded Activities.
- Recovery is **at-least-once**, so every irreversible side effect needs an **idempotency key** derived from stable run/step identity. Enumerating those side effects is a required design artifact.
- Durability buys crash resumption, deploy survival, arbitrary waits, and auditability — at the cost of determinism constraints, hard versioning, and operational surface. **Reserve it for runs that cross process lifetimes or take irreversible actions.**
