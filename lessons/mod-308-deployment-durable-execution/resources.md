# Resources for mod-308-deployment-durable-execution

Primary references for deployment, durable execution, and human-in-the-loop architecture. Verify against current docs — this tooling moves fast.

## Durable execution and resumption

- **Temporal — What is Temporal? / Core concepts** ([docs.temporal.io](https://docs.temporal.io/temporal)) — Workflows, Activities, Workers, and the durable event history. The reference model for the durable/ephemeral split in Chapter 1.
- **Temporal — Workflow determinism & deterministic constraints** ([docs.temporal.io/workflow-definition#deterministic-constraints](https://docs.temporal.io/workflow-definition#deterministic-constraints)) — exactly what workflow code may not do, and why replay requires it. Core to the replay boundary in Chapter 1.
- **Temporal — Activities & at-least-once execution / idempotency** ([docs.temporal.io/activities](https://docs.temporal.io/activities)) — why activities can run more than once and how to make side effects idempotent.
- **Temporal — AI agents & durable execution** ([temporal.io/solutions/ai](https://temporal.io/solutions/ai)) — applying durable execution specifically to long-running, stateful LLM agents.
- **DBOS — Durable execution / transact** ([docs.dbos.dev](https://docs.dbos.dev)) — a lighter-weight, library-style durable-execution substrate; useful contrast to a full workflow engine.
- **AWS Step Functions — Developer Guide** ([docs.aws.amazon.com/step-functions](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)) — managed state-machine orchestration; the "managed step orchestrator" row of the substrate table.

## In-flight-safe deployment

- **Temporal — Worker Versioning & Build IDs** ([docs.temporal.io/production-deployment/worker-deployments/worker-versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)) — pin runs to a build ID and run multiple versions side by side. The reference mechanism for rainbow deploys.
- **Temporal — Workflow versioning / patching** ([docs.temporal.io/develop/python/versioning](https://docs.temporal.io/develop/python/versioning)) — `patch`/`GetVersion` gates for changing in-flight runs without breaking replay.
- **AWS — Blue/Green deployments** ([docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html)) — the baseline strategy and why it assumes short-lived requests.
- **Martin Fowler — BlueGreenDeployment** ([martinfowler.com/bliki/BlueGreenDeployment.html](https://martinfowler.com/bliki/BlueGreenDeployment.html)) — the canonical write-up of blue-green and its cutover semantics.
- **Kubernetes — Rolling updates & pod termination / graceful shutdown** ([kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)) — `terminationGracePeriodSeconds` and draining; why "drain" is seconds, not days.
- **AWS — Deployment strategies overview (rolling, blue/green, canary)** ([docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/welcome.html](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/welcome.html)) — the strategy spectrum the rainbow pattern extends for long-running work.

## Human-in-the-loop

- **LangGraph — Human-in-the-loop** ([langchain-ai.github.io/langgraph/concepts/human_in_the_loop/](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/)) — the approve / edit / reject / respond decision set and the `interrupt` model. Source of Chapter 3's vocabulary.
- **LangGraph — `interrupt` & resuming with `Command`** ([langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/add-human-in-the-loop/](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/add-human-in-the-loop/)) — pausing a graph and resuming it with a human decision.
- **LangGraph — Persistence & checkpointers** ([langchain-ai.github.io/langgraph/concepts/persistence/](https://langchain-ai.github.io/langgraph/concepts/persistence/)) — how interrupt state survives an arbitrary wait and a process restart; the durability under the HITL gate.
- **Temporal — Signals** ([docs.temporal.io/encyclopedia/workflow-message-passing](https://docs.temporal.io/encyclopedia/workflow-message-passing)) — delivering a human decision to a sleeping run as a durable signal; the engine-native way to build the gate.
- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — where human checkpoints fit in the agent/workflow taxonomy; the architectural framing for gate placement.

## Fleet scaling and recovery

- **Temporal — Worker performance & tuning** ([docs.temporal.io/develop/worker-performance](https://docs.temporal.io/develop/worker-performance)) — scaling the stateless worker tier independently of run state.
- **Kubernetes — Horizontal Pod Autoscaler** ([kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)) and **KEDA — event/queue-driven autoscaling** ([keda.sh/docs/latest/concepts/](https://keda.sh/docs/latest/concepts/)) — autoscaling on backlog/queue depth rather than CPU, the right signal for agent fleets.
- **Google SRE Book — Handling Overload & Addressing Cascading Failures** ([sre.google/sre-book/handling-overload/](https://sre.google/sre-book/handling-overload/)) — backpressure, load shedding, and poison-request / cascading-failure dynamics that map directly to poison runs and dependency backpressure.
- **AWS — Reliability Pillar (Well-Architected): failure management & multi-AZ** ([docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)) — multi-AZ durability and recovery design behind the zone-failure row of the recovery matrix.
