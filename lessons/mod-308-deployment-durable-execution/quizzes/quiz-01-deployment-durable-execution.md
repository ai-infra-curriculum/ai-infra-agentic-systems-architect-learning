# Quiz 1 — Deployment, Durable Execution & Human-in-the-Loop

Covers all four chapters of [mod-308](../README.md). Twelve questions: answer, then check against the key at the end. Aim to *explain* each answer at the architecture altitude, not just pick a letter.

---

## Questions

### 1. The core split

A long-running agent's progress must survive a process death. Which split does durable execution draw to make that possible?

- A. Read replicas vs. write primary
- B. Durable plane (workflow state) vs. ephemeral plane (workers/activities)
- C. Frontend vs. backend
- D. Control plane vs. data plane in the networking sense

### 2. Why deterministic replay constrains workflow code

Durable engines like Temporal resume a run by **replaying** the workflow function against its recorded event history. What does this require of the workflow code?

- A. Nothing special — replay snapshots memory
- B. The workflow must be deterministic: no wall-clock reads, randomness, or network calls inside it
- C. The workflow must be written in a compiled language
- D. The workflow must run on a single dedicated worker forever

### 3. Where non-determinism lives

In the durable model, where does an agent's **LLM call** belong, and why?

- A. In the workflow, because it is the most important step
- B. In an activity, because its output is non-deterministic and must be *recorded* so replay returns the same result
- C. Nowhere — LLM calls cannot be made durable
- D. In the durable store directly

### 4. At-least-once and idempotency

A worker completes an activity that charges a card, then dies *before* reporting completion to the durable plane. What happens, and what must the design guarantee?

- A. The run is lost; nothing can be done
- B. The engine reschedules the activity (so it may run twice); the side effect must be **idempotent** via a stable key so "ran twice" equals "ran once"
- C. The card charge is automatically refunded
- D. The durable plane blocks all other runs until a human intervenes

### 5. Idempotency key derivation

What must an idempotency key be derived from?

- A. The current wall-clock time
- B. A fresh UUID generated at call time
- C. Stable run/step identity (e.g., a hash of run_id, step_id, payload)
- D. The worker's hostname

### 6. Why rolling/blue-green break long runs

Why does a standard rolling deploy with a 30-second drain timeout sever a multi-day agent run?

- A. Rolling deploys corrupt the database
- B. "Drain" waits only seconds for in-flight work; a multi-day run cannot drain in time, so the deploy kills its worker
- C. Rolling deploys require downtime
- D. Long runs are incompatible with containers

### 7. The rainbow deployment rule

What is the defining rule of a rainbow (versioned, never-replace) deployment?

- A. Always delete the old version immediately to save resources
- B. Never delete a version that still has in-flight runs; deploy the new version alongside the old ones and pin new runs to the newest
- C. Deploy only on weekends
- D. Run exactly two versions at all times

### 8. Pin vs. patch

In a rainbow deploy, when do you **patch** (force a change into in-flight runs) rather than **pin** (let old runs finish on old code)?

- A. Always patch — pinning is never safe
- B. Patch by default; pinning is the exception
- C. Pin by default; patch only for forced changes (e.g., a security fix or a broken downstream contract) that cannot wait for runs to drain
- D. Patch only on the first deploy of the day

### 9. HITL as a durable interrupt

Why is a blocking `input("approve?")` call the wrong way to implement a human-in-the-loop gate?

- A. It is too slow to render
- B. It holds a process/thread open for the wait; if a deploy or crash kills the process mid-wait, the run and the pending approval are lost
- C. It cannot display the proposed action
- D. It always auto-approves

### 10. The four-action decision set

A human reviewing a proposed action chooses **edit**. What does that resume the run with?

- A. The original action, unchanged
- B. The action executed with **human-modified arguments**, treated as authoritative
- C. A rejection that sends control back to the agent
- D. A free-text response injected as an observation

### 11. The scaling signal for agent fleets

Your fleet is 90% sub-minute runs and 2% multi-day approvals. Why is **concurrent run count** the wrong autoscaling signal, and what is right?

- A. Run count is fine; use it
- B. Sleeping approval runs inflate the count but consume ~zero compute; autoscale on **ready-step backlog bounded by downstream rate limits** instead
- C. Autoscale on CPU only
- D. Never autoscale; fix the worker count

### 12. Recovery across scopes

With durable execution in place, what is the recovery action when a worker (or a whole zone) fails mid-run?

- A. Restart the run from the beginning
- B. Reschedule the unfinished step onto another worker (in a surviving zone); the run resumes from the last recorded step — provided the durable store is HA, workers span zones with no affinity, and steps are idempotent
- C. Wait for the failed worker to come back
- D. Drop the run and notify the user

---

## Answer key

1. **B.** The durable plane owns *what has happened and what happens next* (workflow state, persisted); the ephemeral plane owns *doing the next thing* (stateless workers/activities). Decoupling the two is what lets a killed worker not kill the run. (Chapter 1)

2. **B.** Engines replay the workflow against its event history rather than snapshotting memory, so the code must produce the same step sequence every replay — which forbids wall-clock reads, randomness, and network calls inside the workflow. (Chapter 1)

3. **B.** An LLM call is non-deterministic, so it must be an activity whose *result* is recorded; on replay the recorded result is returned instead of re-calling the model. This maps the agent's control flow → deterministic workflow, every model/tool call → recorded activity. (Chapter 1)

4. **B.** Recovery is *at-least-once*: the engine reschedules the unreported activity, so it can run twice. The architect must make every irreversible side effect idempotent so a re-run is harmless. Enumerating those side effects is a required design artifact. (Chapter 1)

5. **C.** The key must come from stable run/step identity so the *same* logical step always produces the *same* key. Deriving it from wall-clock time or a fresh UUID would make every retry look new and defeat deduplication. (Chapter 1)

6. **B.** For a web service, "drain" means waiting seconds for in-flight HTTP requests; for an agent, draining a multi-day run would take days, so standard rolling deploys (with second-scale timeouts) kill the worker instead. Raising the timeout to days would mean the deploy never finishes. (Chapter 2)

7. **B.** Rainbow deploys never remove a version with live runs; the new version runs alongside the old, new runs pin to the newest, and old runs finish on theirs. Temporal's Worker Versioning / Build IDs is the reference mechanism. (Chapter 2)

8. **C.** Pinning is simpler and replay-safe and should be the default; patching (version-guarding a changed branch so old runs replay the old path) is reserved for forced changes that cannot wait for runs to drain. (Chapter 2)

9. **B.** A blocking call ties the wait to the process lifetime. The gate must instead be a durable interrupt: the run persists "waiting at gate G," yields its worker, and resumes on a signal — so the wait survives an arbitrary delay and a process restart. (Chapter 3)

10. **B.** Edit executes the action with human-modified arguments, which the agent must treat as authoritative. (Contrast: approve = run as-is; reject = return control to replan; respond = inject a free-text observation and continue.) (Chapter 3)

11. **B.** Sleeping runs consume ~zero worker capacity (they are rows in the durable store), so counting them over-provisions. The right signal is ready-step backlog (or schedule-to-start latency), capped by downstream rate limits so you don't scale past what the LLM/tool APIs can absorb. (Chapter 4)

12. **B.** Because progress is durable, recovery at every scope is "reschedule the unfinished step," not "restart the run." This holds only if the durable store is multi-AZ HA, workers span zones with no run affinity, steps are idempotent, and dependency failures pause runs durably rather than crash them. (Chapter 4)

---

## Scoring

- **11–12:** Strong. You can reason about durability, deploys, HITL, and fleet recovery as one design problem.
- **8–10:** Solid. Revisit any chapter whose questions you missed.
- **Below 8:** Re-read the chapters, focusing on *why* each mechanism exists, not just what it is — the exercises assume the why.
