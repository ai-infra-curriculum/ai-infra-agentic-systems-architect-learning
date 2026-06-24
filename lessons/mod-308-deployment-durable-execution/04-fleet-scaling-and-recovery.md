# Chapter 4 — Scaling and Failure Recovery for Agent Fleets

The first three chapters secured a *single* long-running run: it survives a crash (Chapter 1), a deploy (Chapter 2), and a human wait (Chapter 3). This chapter zooms out to the **fleet** — thousands of these runs at once — and asks the two operations questions an architect owns: *how do you size and scale the compute*, and *what happens when a worker, a zone, or a downstream dependency fails?* The good news: the durable-execution split from Chapter 1 makes both answerable. The bad news: agent fleets have a workload shape that breaks the autoscaling instincts you brought from web services.

## Scale the workers, not the runs

The foundational move repeats the Chapter 1 split, now as a scaling principle:

```text
   DURABLE PLANE              EPHEMERAL PLANE
   ─────────────              ───────────────
   run state for ALL          worker pool: stateless processes
   in-flight runs    ◀──────  that pull the next ready step,
   (persisted; the            execute it, report back, repeat.
    "source of truth")
                              SCALE THIS POOL UP AND DOWN FREELY —
   sized by # of              no run state lives here, so adding
   live runs, NOT by          or removing a worker never moves
   worker count               or loses a run.
```

Because runs live in the durable plane, **the worker pool is a stateless, freely-scalable compute tier**. You can double it during a spike and halve it overnight; no run is pinned to a specific worker, so scaling never strands or loses work. This is the single biggest architectural payoff of durable execution at fleet scale: capacity planning for *compute* is decoupled from the population of *runs*. A run that is asleep on a timer or a human approval consumes **zero worker capacity** — it is just a row in the durable store — so 10,000 runs waiting on approvals cost almost nothing to hold.

The unit you actually scale on is therefore **ready steps per second**, not concurrent runs. A million runs that are all asleep need a handful of workers; a thousand runs all hammering tools at once need many. Size the pool to the *active-step throughput*, and let the durable plane hold the rest.

## The workload shape: spiky, long-tailed, and bursty per run

Web-service autoscaling assumes roughly uniform, short requests; CPU or request-rate tracks load well. Agent fleets violate every part of that assumption, and an architect must design around three properties:

- **Bimodal, long-tailed run duration.** Most runs finish quickly; a few run for hours or days (and a few sleep on approvals for a week). Average duration is a lie; you must plan for the tail. A p99 run that lives 100x the median means your "drain before deploy" and "recover after failure" timelines are governed by the tail, not the mean.
- **Bursty step demand within a run.** A single run alternates between *waiting* (on an LLM, a tool, a human — zero compute) and *bursting* (parallel tool fan-out — heavy compute). Fleet load is the sum of these sawtooths and is far spikier than request count suggests.
- **Backpressure from shared dependencies.** Every active step likely calls an LLM API or a tool with its own rate limit. Scaling workers past what those dependencies can absorb just converts "slow" into "rate-limited errors." Worker capacity and dependency capacity must be sized **together**.

```text
   naive signal (concurrent runs)   ──▶ misleads: most runs are asleep
   better signal (ready steps/sec)  ──▶ tracks real compute demand
   guardrail (dependency rate limit)──▶ caps useful worker count

   autoscale target ≈ min( steps/sec demand,
                           dependency-rate-limit / steps-per-call )
```

The design rule: **autoscale on queue depth of ready steps (or task-schedule-to-start latency), bounded by downstream rate limits.** Scaling on CPU or raw run count will either over-provision (counting sleeping runs) or thrash (missing the bursts).

## Capacity planning for the long tail

Three concrete sizing artifacts an architect should produce:

- **Worker pool floor and ceiling.** The floor covers steady-state ready-step throughput plus headroom for a burst; the ceiling is set by downstream rate limits and budget (mod-307). Autoscale between them on ready-step backlog.
- **Concurrency caps per dependency.** A global limit on in-flight calls to each LLM/tool so a fleet-wide burst cannot blow a shared rate limit. This is fleet-level backpressure: when the cap is hit, ready steps queue in the durable plane (safe — they are persisted) rather than failing.
- **Tail budget.** An explicit policy for the longest-lived runs: a max run age / approval TTL (from Chapter 3) so the tail is *bounded*, and reserved capacity so a wave of tail runs resuming at once (e.g., a batch of approvals all clicked Monday morning) does not starve fresh runs.

Note how these reuse earlier chapters: the TTL that bounds the tail here is the same TTL that bounds rainbow version sprawl in Chapter 2 and times out an approval in Chapter 3. One policy, three payoffs.

## Failure recovery: a layered model

Failures come at three scopes, and the recovery story differs at each. The architect's deliverable is a **recovery matrix** that names the failure, the detection signal, the recovery action, and the resulting guarantee.

```text
   scope        failure                 recovery (because state is durable)
   ──────────   ─────────────────────   ─────────────────────────────────────
   WORKER       pod OOM / crash /        engine sees missing heartbeat → reschedules
                spot reclaim             the in-flight step onto another worker;
                                         run resumes from last recorded step.
                                         Idempotency (Ch.1) makes a re-run safe.

   ZONE / AZ    whole availability       workers in surviving zones pull the
                zone goes dark           orphaned runs (durable store is
                                         multi-AZ replicated); runs resume.
                                         Requires: store HA, workers spread
                                         across zones, no zone affinity on runs.

   DEPENDENCY   LLM/tool API down,       Activity-level retry with backoff;
                rate-limited, timing     circuit-break to a fallback model/path;
                out                      if unrecoverable, the run PAUSES durably
                                         (not crashes) and resumes when the
                                         dependency returns or a human intervenes.
```

The throughline: **because progress is durable, recovery is "reschedule the unfinished step," not "restart the run."** That is true at every scope. What the architect must guarantee for it to hold:

- **The durable store is itself highly available** (multi-AZ replicated, backed up). It is the single source of truth; if it dies, the whole fleet's state dies with it. This is the one component you cannot make stateless, so it gets the strongest availability investment.
- **Workers are spread across zones** and runs carry **no zone affinity**, so a surviving zone can pick up an orphaned run.
- **Steps are idempotent and retryable** (Chapter 1), so a step that gets re-run after a worker death does not double its side effects.
- **Dependency failures pause, not crash.** A long run should treat a down LLM API as a retryable Activity failure that backs off and eventually pauses the run durably — not as a fatal error that loses the run. The run waits in the durable plane (cheap) until the dependency recovers.

## Poison runs and blast-radius control

Two fleet-specific failure modes have nothing to do with infrastructure and must be designed for:

- **Poison runs.** A single run that deterministically crashes its worker (a malformed input, a replay-divergence bug) will be rescheduled, crash the next worker, and so on — taking down workers fleet-wide. The defense is a **per-run failure budget**: after N consecutive step failures, the engine quarantines the run (moves it to a dead-letter / manual-review state) instead of endlessly rescheduling it. Without this, one bad run can degrade the whole fleet.
- **Runaway fan-out.** A run that spawns unbounded sub-runs or tool calls (the cost-incident failure mode from mod-302/mod-307) can exhaust the pool. Enforce **per-run resource caps** (max concurrent activities, max total steps, max spend) so one run's blast radius is bounded to itself.

Both are about **isolation**: a single misbehaving run must not be able to consume the fleet's capacity or crash its workers. Quarantine and per-run caps are the mechanisms.

## Worked judgment

*"We run 50,000 agent runs/day; 90% finish in under a minute, but ~2% wait on multi-day human approvals."* — The 2% would dominate a run-count autoscaler with sleeping runs. **Autoscale on ready-step backlog, not run count**; the sleeping approvals cost ~nothing in the durable plane. Add an approval TTL to bound the tail and reserved capacity for the Monday-morning approval wave.

*"An AZ outage took out a third of our workers and our agents all errored out."* — They errored because run state lived in the workers. **Move state to a multi-AZ durable plane, spread workers across zones, remove zone affinity**; then an AZ loss becomes "surviving zones pick up the orphaned runs," not a fleet-wide outage.

## Key takeaways

- The durable/ephemeral split makes the **worker pool a stateless, freely-scalable tier**: scale compute independently of the run population, and sleeping runs cost ~zero capacity.
- Agent fleets are **spiky, long-tailed, and bursty per run**. Autoscale on **ready-step backlog bounded by downstream rate limits** — not CPU or concurrent-run count, which over-provision or thrash. Size worker and dependency capacity together.
- Recovery at every scope (worker / zone / dependency) is **"reschedule the unfinished step," not "restart the run"** — provided the durable store is HA, workers span zones with no affinity, steps are idempotent, and dependency failures *pause* runs durably rather than crash them.
- Isolate misbehaving runs: a **per-run failure budget** quarantines poison runs before they take down the fleet, and **per-run resource caps** bound runaway fan-out to a single run's blast radius.
