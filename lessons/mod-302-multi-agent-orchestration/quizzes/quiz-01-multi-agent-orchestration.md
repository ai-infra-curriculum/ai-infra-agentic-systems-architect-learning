# Quiz 1 — Multi-Agent Orchestration Architecture

Knowledge check for [mod-302](../README.md). Answers are at the bottom; try each question before scrolling. Covers all four chapters.

## Questions

### 1. The multiplier

Anthropic's multi-agent research system reports a single agent uses roughly 4x the tokens of a chat. What is the reported multiplier for a *multi-agent* system, and is it something you optimize away?

- A. ~8x, and yes — better prompts remove most of it.
- B. ~15x, and no — it's structural (context duplication, orchestrator round-trips, wide tool surfaces).
- C. ~2x, and it's a measurement artifact.
- D. ~50x, and only hierarchy causes it.

### 2. The dominant cost lever

In the orchestrator-worker token envelope, which design choice most controls how fast the orchestrator's context — and cost — grows with N workers?

- A. The model temperature.
- B. Whether workers return distilled summaries vs. full transcripts.
- C. The order workers are dispatched in.
- D. The orchestrator's system prompt length.

### 3. When fan-out is the wrong shape

You have sub-tasks where each depends on the previous one's output, in sequence. What does an orchestrator fan-out cost you here?

- A. Nothing; fan-out handles dependencies natively.
- B. It wastes tokens re-establishing context because the work isn't independent — a chain/handoff fits better.
- C. It runs faster but costs more.
- D. It forces hierarchy.

### 4. Hierarchy's cost

Why is a hierarchical topology (orchestrator → sub-orchestrators → workers) something to reach for sparingly?

- A. It can't return distilled results.
- B. Each sub-orchestrator is itself a ~15x system nested in a worker slot, so the multiplier compounds.
- C. Frameworks don't support it.
- D. It removes all parallelism.

### 5. Answer ownership

Under the **manager-as-tools** delegation style, who owns the final user-facing answer?

- A. Whichever specialist was called last.
- B. The manager, always — specialists return results to it and never speak to the user.
- C. Ownership is shared across all specialists.
- D. The user.

### 6. Handoff cost

A team picks **handoffs** (control transfer) over manager-as-tools. What is the main safety cost they take on?

- A. Higher token spend in the manager.
- B. Guardrails become diffuse — every specialist that can hold the conversation must enforce them, because any can be the user-facing voice.
- C. Specialists lose all autonomy.
- D. The user hears a single consistent voice, which is bad.

### 7. Orphaned ownership

After a handoff, no agent believes it owns the answer and the user is left hanging. What's the design rule that prevents this?

- A. Disable handoffs entirely.
- B. Every handoff must name the new owner in its transfer payload.
- C. Add more specialists.
- D. Let the user re-ask.

### 8. MCP vs. A2A — the test

You're deciding whether a connection should use MCP or A2A. What's the discriminating test from Chapter 3?

- A. Whichever protocol the framework defaults to.
- B. If the other side could be a deterministic function, it's a tool (MCP); if it must reason/plan with its own tools, it's an agent (A2A).
- C. MCP for internal, A2A for anything over HTTP.
- D. Always A2A — it's the newer standard.

### 9. Where A2A earns its weight

Inside a single system, most "agent-to-agent" needs are really orchestrator-to-worker calls. Where does A2A genuinely pay for its heavier task lifecycle?

- A. Never; A2A is always overhead.
- B. At organizational/vendor boundaries, where the peer is a black box owned by another team and you depend on its capability card, not its internals.
- C. Only for read-only tools.
- D. Whenever latency matters.

### 10. Bounding loops

In an escalation strategy, what should happen when a loop bound (turns, handoffs, iterations, spend, wall-clock) is tripped?

- A. The system crashes with an error.
- B. The bound is ignored and the loop continues.
- C. The system steps up the escalation ladder (degrade, human, or safe failure) — it doesn't crash.
- D. The orchestrator fabricates a best-guess answer to finish.

### 11. The no-fabrication invariant

Why must an escalation ladder always include "admit the gap" rungs (degrade / human / safe-failure)?

- A. To satisfy logging requirements.
- B. So the model has a sanctioned way to *not* know, instead of inventing a plausible answer when it can't make real progress.
- C. To reduce token cost.
- D. Because frameworks require five rungs.

### 12. Partial failure

In a 5-worker fan-out, one worker fails. What's the architecturally correct outcome?

- A. Crash the whole run; a partial answer is worthless.
- B. Synthesize from the 4 successes and silently drop the missing piece.
- C. Synthesize from the 4 successes and explicitly flag the gap ("4 of 5 sources; vendor X unavailable").
- D. Retry the failed worker forever until it succeeds.

## Answer key

1. **B** — ~15x, and it's structural: per-worker context duplication, orchestrator round-trips, and wide tool surfaces ([Chapter 1](../01-orchestrator-worker-topologies.md)).
2. **B** — The distilled-return ratio governs orchestrator-context growth; transcripts make it explode superlinearly ([Chapter 1](../01-orchestrator-worker-topologies.md)).
3. **B** — Dependent sequential work isn't independent breadth; fan-out wastes tokens re-establishing context, so a chain/handoff fits ([Chapter 1](../01-orchestrator-worker-topologies.md)).
4. **B** — A sub-orchestrator is a ~15x system nested in a worker slot, so the multiplier compounds; use hierarchy only when one decomposition layer can't express the task ([Chapter 1](../01-orchestrator-worker-topologies.md)).
5. **B** — Manager-as-tools keeps the manager in control; it always owns the answer and is the only user-facing voice ([Chapter 2](../02-handoffs-vs-manager-as-tools.md)).
6. **B** — With control transfer, any specialist can be the voice, so guardrails must be replicated into each one — diffuse enforcement ([Chapter 2](../02-handoffs-vs-manager-as-tools.md)).
7. **B** — Every handoff must name the new owner; an unnamed transfer is how answers get orphaned ([Chapter 2](../02-handoffs-vs-manager-as-tools.md)).
8. **B** — Could-it-be-a-function is the test: function → MCP capability; must reason/plan with its own tools → A2A agent ([Chapter 3](../03-inter-agent-protocols.md)).
9. **B** — A2A earns its weight at organizational/vendor boundaries where the peer is opaque and you depend on the capability card ([Chapter 3](../03-inter-agent-protocols.md)).
10. **C** — A tripped bound steps up the escalation ladder; it does not crash the system ([Chapter 4](../04-coordination-and-escalation.md)).
11. **B** — The "admit the gap" rungs give the model a sanctioned way to not know, preventing fabrication ([Chapter 4](../04-coordination-and-escalation.md)).
12. **C** — Graceful degradation: synthesize from the successes and flag the gap honestly; never crash and never silently drop ([Chapter 4](../04-coordination-and-escalation.md)).
