# Chapter 2 — In-Flight-Safe Deployment Strategies

You ship a code change to your agent platform on a Tuesday afternoon. Somewhere in the fleet, 4,000 agent runs are mid-flight — some on step 2, some on step 40, some sleeping for a day on a human approval. A deploy strategy that was perfectly safe for a stateless web service can **sever every one of those runs**. This chapter is about why the standard deploy patterns break long-running agents, and how to deploy without cutting in-flight work.

The constraint that makes agent deploys different from web-service deploys is the one from [Chapter 1](01-durable-execution-and-resumption.md): a run can be *in progress* for hours or days, and that in-progress state may be tied to a specific version of the code. Replace the code carelessly and you either kill the run or — worse — resume it on code that no longer matches its recorded history.

## Why blue-green and rolling deploys break long runs

The two default strategies most teams reach for both assume requests are short-lived. That assumption is exactly what fails here.

**Blue-green** stands up a full new ("green") environment, cuts all traffic over, and tears down the old ("blue") one:

```text
   before cutover            after cutover
   ──────────────            ─────────────
   [ BLUE  v1 ] ◀─traffic    [ BLUE  v1 ]  ◀─ TORN DOWN
   [ GREEN v2 ]              [ GREEN v2 ] ◀─traffic

   problem: every long run still executing on BLUE v1
            is killed when BLUE is torn down.
```

For a stateless service the cutover is instant and safe — in-flight requests finish in milliseconds. For an agent fleet, the runs on BLUE are *not* finished; tearing BLUE down mid-flight destroys their workers. Even with durable execution, those runs now have *no v1 worker* to resume on, and if v2 changed the workflow's step sequence, they cannot validly resume on v2 either.

**Rolling deploys** replace pods incrementally — drain a pod, kill it, start a v2 pod, repeat:

```text
   pod-1 v1  ─drain&kill→  pod-1 v2
   pod-2 v1  ─drain&kill→  pod-2 v2     each kill evicts whatever
   pod-3 v1  ─drain&kill→  pod-3 v2     long runs were on that pod
```

Rolling is gentler — it does not tear everything down at once — but "drain" for a web service means "wait for in-flight HTTP requests to finish," which takes seconds. For an agent, draining a pod that hosts a three-day run means *either* waiting three days (you cannot — the deploy would never finish) *or* killing the run. Standard rolling deploys pick "kill," because their drain timeout is measured in seconds, not days.

The root cause in both cases: **these strategies couple the lifetime of the run to the lifetime of the process, and then replace the process.** Durable execution decouples *state* from the process — but you still must decouple *deployment* from the process, or the deploy re-couples them by killing workers that have nowhere to resume.

## The fix, part one: durable execution makes workers replaceable

The first half of the solution comes free from [Chapter 1](01-durable-execution-and-resumption.md). If progress lives in the durable plane, killing a worker does not kill the run — the engine reschedules the unfinished step onto another worker. So a rolling deploy of *workers* is safe **as long as some worker can still run the version of the code the run needs.** A run started on v1 must be able to resume on *a v1 worker* (or on v2 code that is replay-compatible with v1's history).

That last clause is the whole problem. Two failure modes remain even with durable execution:

- **No compatible worker.** You replaced all v1 workers with v2 workers, and v2 is not replay-compatible with v1 histories. In-flight v1 runs are now stranded — they have state but no code that can validly advance it.
- **Replay divergence.** v2's workflow code takes a different sequence of steps than v1 recorded. On replay, the engine sees the recorded history disagree with the live code path and raises a non-determinism error. The run is poisoned.

## The fix, part two: rainbow deployments

A **rainbow deployment** (also called a *versioned* or *roll-forward, never-replace* deployment) solves both. The rule is simple:

> **Never delete a version that still has in-flight runs.** Deploy the new version alongside all older versions, route new runs to the newest, and let each old version's workers stay alive until the last run on that version drains.

```text
   ┌──────────────────────────────────────────────────────────┐
   │  RAINBOW: every live version coexists                     │
   │                                                           │
   │  v1 workers ── serving runs r17, r88   (draining)         │
   │  v2 workers ── serving runs r402…r900  (draining)         │
   │  v3 workers ── serving all NEW runs    (active)           │
   │                                                           │
   │  new run ──▶ always pinned to NEWEST version (v3)         │
   │  old run ──▶ stays on the version it STARTED on           │
   │                                                           │
   │  retire v1 ONLY when r17 and r88 have completed.          │
   └──────────────────────────────────────────────────────────┘
```

Each color in the rainbow is a complete, independently-scalable deployment of one code version. New runs are **version-pinned** at start to the newest color. Old runs ride their original color to completion. A color is decommissioned only when its run count reaches zero. Because long runs eventually finish (or are aged out — see below), old colors drain over time and the rainbow stays bounded.

Temporal formalizes this as **Worker Versioning / Build IDs**: each worker advertises a build ID, the server pins each run to a build ID, and it dispatches tasks only to compatible workers. You can also achieve the pattern without a workflow engine, by tagging deployments with a version and routing started runs to the matching deployment — the *mechanism* differs, the *contract* is identical.

## Where rainbow meets workflow versioning

Rainbow deploys solve the *infrastructure* half (keep old code running). The *code* half is keeping replay valid when you genuinely need a run to migrate onto new logic. Two complementary tools:

- **Version pinning (the default).** A run finishes on the code it started on. Zero replay risk, at the cost of running multiple versions concurrently. Use this as the baseline.
- **Patching / `GetVersion` gates (when you must change in-flight runs).** When a bug fix or a required change must reach already-running runs, you guard the changed branch with a version marker recorded in history, so old runs replay the old path and new runs take the new path. This lets one worker validly serve both — at the cost of accumulating version-guard code you must eventually clean up.

The architectural decision is: *do you let old runs finish on old code (pin), or migrate them onto new code (patch)?* Pinning is simpler and safer and should be the default; patching is for forced changes — a security fix, a broken downstream contract — that cannot wait for runs to drain.

## The drain problem and aging out

Rainbow deploys assume old runs eventually end. Two things threaten that:

- **Runs that wait indefinitely.** A run sleeping on a human approval ([Chapter 3](03-human-in-the-loop-architecture.md)) can hold its version alive for weeks. You need a policy: a **maximum run age / approval TTL** after which a run is escalated, cancelled, or force-migrated, so old colors cannot live forever.
- **Version sprawl.** Deploy ten times in a day with slow-draining runs and you have ten live colors, each consuming capacity. Bound it with a **version-retention policy** (e.g., "support at most N concurrent versions; if a deploy would exceed N, migrate the oldest version's runs via patching before proceeding").

These policies are the architect's deliverable. The deploy *mechanism* keeps old runs alive; the *policies* keep the set of live versions bounded so the platform does not accrete colors without end.

## Putting it together: a deployment runbook shape

A safe agent-platform deploy is a sequence, not a button:

```text
  1. Build v_next; tag workers with build ID v_next.
  2. Deploy v_next workers ALONGSIDE current versions (do not remove any).
  3. Server pins all NEW runs to v_next; existing runs stay on their version.
  4. Monitor: in-flight count per version, error rate per version,
     replay-failure count (must stay zero).
  5. If healthy: let old versions DRAIN naturally; retire a version when
     its in-flight count hits 0 (or its run-age TTL forces migration).
  6. If unhealthy: stop pinning new runs to v_next, route them back to the
     previous version (old workers are still alive — instant rollback),
     investigate, retire v_next.
```

Note the rollback property: because rainbow deploys *never remove the previous version*, rollback is "route new runs back to the still-running previous color" — instant and lossless, not a frantic redeploy. That is a direct payoff of never-replace.

## Worked judgment

*"Our agents are stateless, sub-30-second classifiers behind a queue."* — In-flight work finishes in seconds. **Rolling or blue-green is fine**; rainbow is unnecessary complexity. Match the strategy to the run lifetime.

*"Our agents run multi-hour workflows with payment side effects and day-long human approvals."* — A rolling deploy would sever runs; blue-green would mass-kill them. **Rainbow deploy with version pinning** is mandatory; add an approval TTL so old versions drain, and a patching path for forced security fixes.

## Key takeaways

- Blue-green and rolling deploys assume short requests; they **sever long runs** by tearing down or draining the processes those runs live on.
- Durable execution makes workers *replaceable* but not *removable* — a run must always have a **version-compatible worker** to resume on, or it strands / poisons on replay.
- **Rainbow deployments** (never-replace, version-pinned) keep every live version running until its runs drain; new runs go to the newest version, old runs finish on theirs. Temporal's Worker Versioning is the reference mechanism.
- Pin by default; **patch** only for forced changes to in-flight runs. Bound the rainbow with **run-age TTLs** and a **version-retention policy**, and you get instant, lossless rollback for free.
