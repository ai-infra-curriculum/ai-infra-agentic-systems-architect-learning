# Capstone: End-to-End Agentic Reference Architecture

**Duration**: 30 hours | **Difficulty**: Very High | **Project ID**: project-301-production-agentic-reference-architecture

## Overview

This is the integrative capstone for the Agentic Systems Architect track. You will design — and prototype the orchestration of — a production-grade agentic system **of your own choosing**, then defend it as a coherent reference architecture. The system must exercise every capability the track has taught: an orchestrator-workers topology, deliberate memory and context engineering, a trajectory and tool-call evaluation harness, distributed tracing, two-sided guardrails, a cost and latency model, durable execution, human-in-the-loop checkpoints, and a horizontal governance specification.

The deliverable is an **architecture package**, not a finished product. You are graded on the quality of your decisions, the rigor of your trade-off analysis, and whether a working prototype demonstrates that the critical paths are real rather than hand-waved. Roughly one third of the effort is a runnable orchestration spike; the rest is architecture artifacts.

### Domain is learner-selected

You pick the domain. There is no pre-anchored sector, employer, or product. Choose a system you can reason about deeply and that genuinely needs multiple agents and tools. Examples (illustrative, not prescriptive):

- A deep-research agent that plans, fans out retrieval, and synthesizes cited reports.
- A data-operations agent that triages alerts, runs diagnostic queries, and proposes remediations.
- A customer-facing assistant that answers from a knowledge base and takes bounded actions.
- An internal operations agent (finance close, supply-chain exceptions, content moderation, and so on).

Your domain choice is itself an architectural decision and must be justified in ADR-001 (see Deliverables). Pick a domain where the agentic topology earns its complexity — if a single prompt-and-response would suffice, the domain is too simple for this capstone.

## Learning Objectives

By completing this project, you will:

1. Compose every module of the track into one defensible end-to-end architecture.
2. Choose and justify an agent topology (orchestrator-workers vs. alternatives) against your domain.
3. Engineer memory and context as first-class subsystems with explicit retention and retrieval policies.
4. Build an evaluation harness that scores trajectories and tool calls, not just final answers.
5. Make latency, cost, reliability, and safety trade-offs explicit and measurable.
6. Produce architecture artifacts — ADRs, a reference diagram, an eval plan, a cost model, and a threat model — that a senior reviewer can act on.

## Project Scenario

### Context

You are the founding agentic-systems architect for a system you are standing up from scratch. You have a green-field mandate but real constraints: a fixed monthly inference budget, a latency budget that users will notice if you miss, an on-call rotation that must be able to operate the system, and a governance reviewer who will not approve a launch without a threat model and an eval gate.

You do **not** have unlimited model spend, a dedicated SRE team, or the luxury of "we'll add observability later." The architecture must be operable on day one by a small team, and every expensive capability (large-context calls, wide tool fan-out, multi-step retries) must be justified against the cost and latency model.

### Your mission

Design the complete reference architecture, prototype the orchestration core to prove the critical paths, and package the decisions so that a reviewer could approve a build and an engineer could implement it.

## Requirements

### Functional requirements

1. **FR-1 — Orchestration topology**: An orchestrator-workers (or justified alternative) design with explicit task decomposition, worker specialization, result aggregation, and a failure-handling path for partial worker failure.
2. **FR-2 — Memory and context engineering**: Distinct short-term (working), long-term (persistent), and retrieved (external knowledge) memory tiers, each with documented write/read/eviction policies and a context-assembly strategy that respects the model context budget.
3. **FR-3 — Tooling layer**: A typed tool interface with at least four tools, including at least one read tool and one state-changing (write) tool, with input validation at every tool boundary.
4. **FR-4 — Evaluation harness**: A harness that scores both end-state quality and the trajectory (planning quality, tool-call correctness, redundant or wasteful steps), runnable against a fixed eval set as a release gate.
5. **FR-5 — Observability**: End-to-end distributed tracing (OpenTelemetry or equivalent) spanning orchestrator, workers, tool calls, and model calls, with spans carrying token, latency, and cost attributes.
6. **FR-6 — Two-sided guardrails**: Input guardrails (prompt-injection, scope, PII) and output guardrails (groundedness, policy, action authorization) with a documented action-confirmation boundary.
7. **FR-7 — Durable execution and human-in-the-loop**: Long-running or high-risk flows survive process restarts (checkpointing/durable workflow) and pause for human approval at defined high-risk steps.
8. **FR-8 — Governance**: A horizontal governance specification covering ownership, change management for prompts/tools/models, audit logging, and a rollback policy.

### Non-functional requirements

1. **Performance**: A stated end-to-end latency budget (p50 and p95) with a per-stage breakdown showing where the budget is spent.
2. **Cost**: A per-task cost target and a monthly envelope; the cost model must show the dominant cost drivers and at least two levers to reduce them.
3. **Reliability**: Defined behavior under model timeout, tool failure, and partial worker failure; no silent failures.
4. **Safety and security**: A threat model covering prompt injection, tool misuse, data exfiltration, and excessive agency, with mitigations mapped to threats.
5. **Operability**: An on-call engineer can answer "what is this run doing and why" from traces and logs alone.

### Constraints

- **Budget**: Treat inference spend as scarce; assume a fixed monthly envelope you define and defend.
- **Latency**: Define a user-facing latency budget; the prototype must demonstrate the hot path stays within it for the common case.
- **Team size**: Operable by a small team — favor simplicity and observability over cleverness.
- **No vendor lock-in by default**: Model and tool interfaces should be swappable; justify any hard dependency.

## Deliverables

Produce the following in this project directory. Suggested layout:

```text
project-301-production-agentic-reference-architecture/
├── README.md                    # This file
├── adrs/                        # Architecture Decision Records (>=10)
├── diagrams/                    # Reference architecture + sequence diagrams
├── eval/                        # Eval plan, eval set, harness spike
├── cost-model/                  # Cost + latency model (spreadsheet or notebook)
├── threat-model/                # Threat model + mitigation mapping
├── governance/                  # Horizontal governance spec
└── prototype/                   # Runnable orchestration spike
```

1. **ADR set (>=10 ADRs)** — One per major decision: domain choice (ADR-001), topology, memory tiers, tool boundary, eval strategy, observability stack, guardrail placement, durable-execution mechanism, human-in-the-loop boundary, and model/provider strategy. Each ADR states context, options considered, decision, and consequences.
2. **Reference architecture diagram** — A C4-style or equivalent diagram (container plus key sequence views) showing orchestrator, workers, memory tiers, tools, guardrails, eval gate, and tracing. Include at least one sequence diagram of the primary happy path and one of a failure/HITL path.
3. **Evaluation plan plus harness spike** — A documented eval plan (what is scored, how, pass thresholds) plus a runnable harness that scores a small fixed eval set on both end-state and trajectory metrics.
4. **Cost and latency model** — A model (spreadsheet or notebook) deriving per-task cost and p50/p95 latency from per-stage token counts and call counts, with sensitivity to the top cost drivers.
5. **Threat model** — STRIDE-style or equivalent, focused on agentic risks (prompt injection, tool misuse, excessive agency, data exfiltration), each threat mapped to a concrete mitigation in the architecture.
6. **Governance specification** — Ownership, change management for prompts/tools/models, audit logging, rollback, and the eval-gate-as-release-control policy.
7. **Orchestration prototype** — A runnable spike (any stack) that demonstrates the orchestrator-workers loop, at least two real tools (one read, one write), tracing, one guardrail, and one HITL checkpoint. It need not be feature-complete; it must prove the critical paths exist.

## Effort breakdown (30 hours)

| Phase | Focus | Hours | Primary deliverable |
|-------|-------|-------|---------------------|
| 1 | Domain selection, scoping, ADR-001, requirements traceability | 4 | ADR-001 + scope note |
| 2 | Topology, memory, and tooling design (ADRs + diagrams) | 7 | Reference diagram + ADRs |
| 3 | Orchestration prototype (workers, tools, tracing, one guardrail, one HITL) | 8 | Runnable spike |
| 4 | Evaluation harness + eval plan | 4 | Eval plan + harness spike |
| 5 | Cost/latency model + threat model | 4 | Cost model + threat model |
| 6 | Governance spec, package review, self-assessment against rubric | 3 | Governance spec + final package |

## Acceptance criteria

- [ ] Domain is learner-selected and justified in ADR-001; the agentic topology is shown to be warranted.
- [ ] At least 10 ADRs, each with context, options, decision, and consequences.
- [ ] Reference diagram covers all eight functional requirements and includes at least two sequence views (happy path + failure/HITL).
- [ ] Memory design names three tiers with explicit write/read/eviction policies and a context-budget strategy.
- [ ] Tooling layer has at least four typed tools including at least one write tool, with validation at every boundary.
- [ ] Eval harness scores both end-state and trajectory and runs as a gate against a fixed eval set.
- [ ] Tracing spans carry token, latency, and cost attributes end to end.
- [ ] Two-sided guardrails are placed in the architecture with a clear action-confirmation boundary.
- [ ] Durable execution survives restart for at least one long-running flow; at least one HITL checkpoint is demonstrated.
- [ ] Cost model derives per-task cost and p50/p95 latency and identifies the top cost drivers and at least two reduction levers.
- [ ] Threat model maps each agentic threat to a concrete mitigation in the design.
- [ ] Governance spec covers ownership, change management, audit logging, and rollback.
- [ ] Prototype runs and demonstrates the orchestrator-workers loop, at least two tools, tracing, one guardrail, and one HITL checkpoint.

### Rubric

| Dimension | Weight | What strong work looks like |
|-----------|--------|------------------------------|
| Architectural soundness | 30% | Topology, memory, and tooling decisions fit the domain; trade-offs are explicit and defensible. |
| Integration completeness | 20% | All eight modules are present and wired together, not bolted on; the diagram and prototype agree. |
| Evaluation rigor | 15% | Trajectory plus end-state scoring with meaningful thresholds; the gate would actually catch regressions. |
| Cost, latency, and reliability | 15% | Quantified budgets, honest sensitivity analysis, defined failure behavior with no silent failures. |
| Safety and governance | 15% | Two-sided guardrails, threat-to-mitigation mapping, and an operable governance lifecycle. |
| Prototype credibility | 5% | The spike demonstrably proves the critical paths rather than mocking them away. |

## Stretch goals

- **Multi-model routing**: Route cheap steps to a small model and hard steps to a large model; show the cost/quality delta in the eval harness.
- **Adversarial eval set**: Add prompt-injection and tool-misuse cases to the eval set and prove the guardrails hold.
- **Replay debugging**: Use traces to reconstruct and replay a failed run deterministically.
- **Budget enforcement**: Make the cost model live — abort or downgrade a run when it exceeds its per-task budget.
- **Chaos drill**: Inject a tool outage during a durable run and show clean recovery from the last checkpoint.

## Submission guidelines

1. Complete all seven deliverables in the directory layout above.
2. Ensure the reference diagram, ADRs, and prototype are mutually consistent.
3. Include a short `RUNNING.md` in `prototype/` so a reviewer can run the spike.
4. Self-assess against the rubric and note any conscious scope cuts in ADRs.

---

**Ready to start?** Begin with Phase 1: pick your domain and write ADR-001.
