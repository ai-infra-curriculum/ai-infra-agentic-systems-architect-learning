# mod-308-deployment-durable-execution: Deployment, Durable Execution & Human-in-the-Loop

**Estimated effort:** 12 hours

An agent that runs for ninety seconds is an API call. An agent that runs for three days — waiting on a tool, a vendor callback, or a human approval — is a **distributed system**, and it will be killed mid-flight by a deploy, an OOM, a spot-instance reclaim, or a node drain long before it finishes. As an **architect**, your job at this layer is not to write the agent loop. It is to decide *how the loop survives the infrastructure it runs on*: where state lives, how a half-finished run resumes after a crash, how you ship new code without severing in-flight work, where a human steps into the loop without losing the run, and how a fleet of these things scales and recovers when a whole availability zone goes dark.

> **Architecture, not mechanics.** The AI-Engineer track teaches you to *build* an agent loop and call a tool. This module pitches up: you choose the durability substrate, draw the deployment topology, write the human-in-the-loop (HITL) approval contract, and design the fleet's failure-recovery behavior. The deliverable is a **design artifact** — a sequence diagram, a state-machine spec, a deployment runbook, a recovery matrix — that an engineer then implements.

The thread running through every chapter: **a long-running agent's state must outlive the process that produced it.** Once you accept that, durable execution, in-flight-safe deploys, persisted HITL gates, and fleet recovery stop being four topics and become four faces of one design problem — separating *what the agent is doing* (durable, replayable) from *where it happens to be running* (ephemeral, replaceable).

## Learning objectives

- **Architect durable execution and resumption** for long-running, stateful agents — separate workflow state from worker compute, make steps replayable, and reason about idempotency and deterministic replay (e.g., Temporal).
- **Design deployment strategies that do not disrupt in-flight agents** — why blue-green and naive rolling deploys break long-running runs, and how versioned, drain-aware patterns (e.g., rainbow deployments) keep old runs alive on old code.
- **Integrate human-in-the-loop approval flows** — the approve / edit / reject / respond decision set — as durable, resumable interrupts whose state survives an arbitrary wait and a process restart.
- **Plan scaling and failure recovery for agent fleets** — capacity for spiky, long-tailed runs; worker autoscaling decoupled from run state; and recovery from worker, zone, and dependency failure.

## Lecture chapters

1. [Durable Execution and Resumption](01-durable-execution-and-resumption.md) — separating workflow state from worker compute, deterministic replay, idempotency, and what "durable" actually buys you.
2. [In-Flight-Safe Deployment Strategies](02-inflight-safe-deployment.md) — why blue-green and rolling deploys sever long runs, and how rainbow / versioned-drain deploys keep them alive.
3. [Human-in-the-Loop Approval Architecture](03-human-in-the-loop-architecture.md) — approve/edit/reject/respond as durable interrupts, state persistence across arbitrary waits, and the authority/audit contract.
4. [Scaling and Failure Recovery for Agent Fleets](04-fleet-scaling-and-recovery.md) — autoscaling decoupled from run state, capacity for long-tailed runs, and worker/zone/dependency recovery.

## Exercises

Hands-on architecture practice — diagrams, state-machine specs, and runbooks, not framework wiring. Reference solutions live in the paired [solutions repo](https://github.com/ai-engineering-curriculum/agentic-systems-architect-solutions).

- [exercise-01: Durable execution design](exercises/exercise-01-durable-execution-design.md) — decompose a long-running agent into durable workflow + replaceable activities and defend the replay/idempotency boundaries.
- [exercise-02: HITL approval architecture](exercises/exercise-02-hitl-approval-architecture.md) — design an approve/edit/reject/respond gate that survives an arbitrary wait and a process restart.
- [exercise-03: Agent fleet deployment strategy](exercises/exercise-03-agent-fleet-deployment-strategy.md) — pick a deploy strategy that never severs an in-flight run and write the runbook.
- [exercise-04: Failure recovery and resumption](exercises/exercise-04-failure-recovery-and-resumption.md) — build a recovery matrix for worker, zone, and dependency failure and prove no run is lost.

## Assessment

- [Quiz 1 — Deployment, Durable Execution & Human-in-the-Loop](quizzes/quiz-01-deployment-durable-execution.md) — covers all four chapters.

## Prerequisites

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — the workflow-vs-agent distinction; durable execution is a workflow substrate.
- [mod-303: Memory & Context Architecture](../mod-303-memory-context-architecture/README.md) — where agent state lives; durability builds on it.
- [mod-307: Cost & Latency Architecture](../mod-307-cost-latency-architecture/README.md) — fleet scaling and recovery trade against the cost envelope.
- Working familiarity with at least one orchestration substrate (Kubernetes, a serverless platform, or a workflow engine) and basic distributed-systems vocabulary (idempotency, at-least-once delivery, draining).

See [resources.md](resources.md) for primary references.
