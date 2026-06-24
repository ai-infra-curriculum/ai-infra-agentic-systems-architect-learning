# Chapter 4 — Coordination and Escalation Strategy

A multi-agent topology that works on the happy path is half a design. The other half is what happens when a worker stalls, two agents contradict each other, a loop won't terminate, or the model hits the edge of its competence. As an architect you own the **coordination strategy** (how agents stay consistent) and the **escalation strategy** (what happens when they can't make progress). These are specifications, not code — an engineer implements your rules.

## Coordination: keeping agents consistent

Independent agents working in parallel will disagree, duplicate work, and step on shared state. The coordination strategy decides how the system stays coherent.

| Mechanism | What it does | When to use |
| --- | --- | --- |
| Shared scratchpad / blackboard | Agents read/write a common state store | Workers that must see each other's partial results |
| Orchestrator-mediated | All cross-agent state flows through the lead | Default; simplest to reason about |
| Explicit message contracts | Typed `{intent, payload, correlation_id}` between agents | Any handoff or A2A delegation |
| Conflict-resolution rule | A defined tie-breaker when results disagree | Workers that can return contradictory findings |

The architect's job is to pick the *minimum* coordination that keeps the system coherent. Orchestrator-mediated state is the default because it keeps reasoning local to the lead; reach for a shared blackboard only when workers genuinely must observe each other mid-flight, and accept that you've added a concurrency surface to govern.

## Escalation: what happens when progress stalls

Escalation is a **ladder**: each rung is a cheaper attempt to recover before paying for the next, ending in a human or a safe failure. Specify the rungs explicitly.

```text
   rung 0  ── retry once (transient error)
   rung 1  ── reassign / re-decompose (bad assignment)
   rung 2  ── degrade: synthesize from partial results, flag the gap
   rung 3  ── escalate to human-in-the-loop (judgment / risk gate)
   rung 4  ── safe failure: return "couldn't complete", never fabricate
```

Two cross-cutting rules govern the whole ladder:

- **Bound every loop.** Every agentic loop — turns per agent, handoffs per request, evaluator-optimizer iterations, total wall-clock and total spend — needs a hard cap. An unbounded loop is a cost and reliability incident waiting to happen. When a bound trips, you don't crash; you step up the escalation ladder.
- **Never fabricate to avoid escalating.** The worst failure mode is an agent that, unable to make real progress, invents a plausible answer. The ladder must always have "admit the gap" rungs (degrade, human, safe-failure) so the model has a sanctioned way to *not* know.

## Deadlock and contention

- **Handoff ping-pong** (A → B → A …). Break with a transfer counter: after K transfers, escalate to a human or to a default owner. Require each handoff to carry forward progress so a repeat transfer is detectable.
- **Mutual wait** (A waits on B, B waits on A). Avoid by keeping a strict delegation hierarchy — children never call parents — or by giving the orchestrator a timeout that forces a decision.
- **Runaway fan-out** (an agent spawns sub-agents that spawn sub-agents). Cap depth and total agent count at the system level, not per agent.

## Human-in-the-loop as an architectural gate

A human gate is a first-class node in the topology, not an exception handler bolted on. Decide *at design time* where one belongs:

- **Risk gate** — before any irreversible or high-cost action (refunds over a threshold, production changes, external sends).
- **Judgment gate** — when confidence is low or agents disagree and the conflict-resolution rule can't break the tie.
- **Review gate** — sampling outputs for quality even on the happy path.

Specify, for each gate: *what triggers it, what the human sees, what the default is on timeout, and who is accountable.* A gate with no timeout default is itself a deadlock.

## Partial failure as the normal case

At scale, *something* always fails. The design target is graceful degradation, not zero failures: one worker erroring should never sink the run. Collect successes, tell the orchestrator which assignments failed and why, and let it synthesize from what came back — explicitly flagging the gap rather than papering over it. "Completed with 4 of 5 sources; pricing for vendor X unavailable" is a good outcome; a confident answer that silently dropped vendor X is a bug.

## Key takeaways

- Coordination keeps parallel agents consistent — prefer orchestrator-mediated state; add a shared blackboard only when workers must see each other mid-flight.
- Escalation is a **ladder**: retry → reassign → degrade → human → safe-failure. Specify the rungs; never let the model fabricate instead of escalating.
- **Bound every loop** (turns, handoffs, iterations, spend, wall-clock); a tripped bound steps up the ladder, it doesn't crash the system.
- Human gates are first-class topology nodes with explicit triggers, defaults, and accountability — and partial failure is the normal case, so design for graceful degradation that flags gaps honestly.
