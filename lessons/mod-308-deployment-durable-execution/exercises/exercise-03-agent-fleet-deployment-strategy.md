# exercise-03: Agent Fleet Deployment Strategy

**Estimated effort:** 3 hours

## Objective

Choose and specify a deployment strategy for an agent platform that **never severs an in-flight run**, then write the runbook an on-call engineer would follow to ship a change safely. The deliverable is a strategy decision (with the rejected alternatives argued down), a versioning policy, and a step-by-step deploy + rollback runbook.

This is an **architecture** exercise. You are designing the deploy *process*, not writing CI/CD YAML.

## Background

This exercise covers material from:

- [Chapter 2 — In-Flight-Safe Deployment Strategies](../02-inflight-safe-deployment.md)
- [Chapter 1 — Durable Execution and Resumption](../01-durable-execution-and-resumption.md) — durable execution makes workers replaceable, which deploys depend on.

## The scenario

You operate a platform running the vendor-onboarding and finance agents from exercises 01–02:

> Most runs finish in minutes, but a meaningful fraction span **multiple days** (waiting on document uploads and human approvals). Runs take **irreversible side effects** (account creation, payments). The team currently ships **3–5 deploys per day**. Today they use a **rolling deploy** with a 30-second drain timeout — and they keep getting paged because long runs error out around deploy windows.

Diagnose why, then design the fix.

## Tasks

### 1. Diagnose the current failure

- Explain precisely why a rolling deploy with a 30-second drain **severs** the multi-day runs, using the mechanics from [Chapter 2](../02-inflight-safe-deployment.md). Why does "drain" mean seconds for a web service but days for an agent — and why can't they just raise the timeout?
- Show what *also* breaks if they switched to blue-green instead.

### 2. Choose the strategy and argue down the alternatives

- Select a deployment strategy that keeps in-flight runs alive. Per [Chapter 2](../02-inflight-safe-deployment.md), this should be a **rainbow / versioned never-replace** deploy.
- Write the argument: why rolling and blue-green are wrong for *this* workload, and what specifically the rainbow pattern provides that they don't.
- Name the concrete mechanism (e.g., Temporal Worker Versioning / Build IDs, or version-tagged deployments with run pinning) and what the server/router must do (pin new runs to the newest version; keep old runs on theirs).

### 3. Write the versioning policy

- Decide your default: **version pinning** (old runs finish on old code) vs. **patching** (force a change into in-flight runs). Per [Chapter 2](../02-inflight-safe-deployment.md), pin by default; specify when patching is justified (a security fix, a broken downstream contract).
- Define the **version-retention policy** (max concurrent live versions; what happens when a deploy would exceed it) and the **run-age TTL** that bounds how long a version can be kept alive by a sleeping run — connecting to the approval TTL from [exercise-02](exercise-02-hitl-approval-architecture.md).

### 4. Write the deploy runbook

Produce an ordered runbook covering:

- Build and tag the new version.
- Deploy **alongside** existing versions (never remove).
- Pin new runs to the newest version; leave existing runs on theirs.
- The monitoring signals to watch (in-flight count per version, error rate per version, **replay-failure count must stay zero**).
- Healthy path: drain and retire old versions when their in-flight count hits 0 (or TTL forces migration).
- Unhealthy path: **instant rollback** by routing new runs back to the still-running previous version.

### 5. Show the rollback property

- Explain why rainbow deploys give **instant, lossless rollback** ("route new runs back to the still-alive previous color") rather than a frantic redeploy, and contrast with rollback under blue-green.

## Starter guidance

The state of the fleet across a rainbow deploy — annotate it with your retirement and TTL rules:

```text
   ┌──────────────────────────────────────────────────────────┐
   │  v1 workers ── runs r17, r88 (draining; TTL clock running)│
   │  v2 workers ── runs r402…r900 (draining)                  │
   │  v3 workers ── ALL new runs (active)                      │
   │                                                           │
   │  new run ──▶ pinned to v3 (newest)                        │
   │  retire v1 ⟺ in-flight(v1) == 0  OR  run-age TTL fires    │
   └──────────────────────────────────────────────────────────┘
```

A runbook skeleton to flesh out:

```text
  1. build v_next; tag workers with build id v_next
  2. deploy v_next ALONGSIDE current versions (remove nothing)
  3. server pins NEW runs → v_next; existing runs stay on their version
  4. watch: in-flight/version, error-rate/version, replay-failures (== 0)
  5. healthy  → let old versions drain; retire when in-flight == 0 (or TTL)
  6. unhealthy→ re-pin NEW runs → previous version (still alive); roll back
```

You do **not** need the failure-recovery matrix (exercise-04) here — note where a deploy-time failure hands off to recovery, but design recovery there.

## Acceptance criteria

You can demonstrate that:

- You correctly diagnose why rolling (and blue-green) severs the multi-day runs, including why raising the drain timeout is not a fix.
- You select a rainbow/versioned never-replace strategy and argue down the alternatives for *this* workload.
- Your versioning policy states pin-by-default, names when patching is justified, and bounds live versions via a retention policy and run-age TTL.
- The runbook is ordered, names the monitoring signals (including replay-failure == 0), and covers both the drain-and-retire and the rollback paths.
- You explain why rollback is instant and lossless under rainbow.

## Reflection

In `NOTES.md`:

1. A run has been asleep on a human approval for 9 days, pinning v1 alive. Walk through what your TTL/retention policy does, and the trade-off you chose (force-migrate via patching vs. let it ride).
2. Give a change that **must** be patched into in-flight runs rather than pinned. How do you guard it so old runs replay the old path and new runs take the new one?
3. With 3–5 deploys/day and slow-draining runs, how many live versions could accumulate, and how does your retention policy keep that bounded?

## Stretch goals

- Add **canary** semantics on top of rainbow: pin only 5% of new runs to v_next, watch its error rate, then ramp. Show how this composes with version pinning.
- Design the **observability** view an on-call engineer needs during a deploy (per-version in-flight, per-version error rate, replay-failure alarm) and where those signals come from.
- Reconcile the deploy strategy with the durable design from [exercise-01](exercise-01-durable-execution-design.md): which workflow-code changes are replay-safe to deploy without patching, and which are not?
