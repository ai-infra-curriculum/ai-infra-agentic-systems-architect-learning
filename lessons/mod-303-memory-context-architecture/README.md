# mod-303-memory-context-architecture: Memory & Context Architecture

**Estimated effort:** 12 hours

Every agent run is an argument with a finite resource: the context window. An ai-engineer learns to *fit* work inside it. An **architect** decides how the system as a whole spends, conserves, and replenishes that resource across thousands of turns and millions of users — what gets loaded just-in-time, what gets compacted away, what is written down to survive a session, and where each kind of state physically lives. This module treats context and memory as a **first-class architectural concern**, not a prompt-tuning afterthought: you'll set token budgets, model and mitigate context rot, draw persistence boundaries between working/episodic/long-term memory, and choose between vector, graph, and hybrid stores on stated trade-offs.

> **Decide, don't tinker.** At L48 the deliverable is rarely a clever prompt. It's a *budget*, a *placement diagram*, and a *decision matrix* a team can build against — with the failure modes named and the trade-offs made explicit. Every chapter ends in an artifact you could hand to engineers.

## Learning objectives

- **Architect context-engineering strategies** — compaction, structured note-taking, sub-agent isolation, and just-in-time retrieval — as a deliberate token budget rather than ad-hoc trimming.
- **Diagnose and mitigate context rot** in long-horizon agents: the measurable degradation that comes with a growing, noisy window, and the architectural levers that hold it back.
- **Decide where state lives** — working, episodic, and long-term memory — and draw the persistence boundaries between them with explicit consistency and privacy properties.
- **Integrate RAG and vector/knowledge-graph stores** as an architectural concern: store selection, retrieval topology, freshness, and how you'll *evaluate* retrieval before it reaches the model.

## Lecture chapters

1. [Context Engineering as a Token Budget](01-context-engineering-strategies.md) — compaction, structured note-taking, sub-agent isolation, and just-in-time retrieval, costed against a finite window.
2. [Diagnosing and Mitigating Context Rot](02-context-rot.md) — why long windows degrade, how to measure it, and the levers that slow it down.
3. [Where State Lives: Memory Tiers and Persistence Boundaries](03-memory-tiers-and-state.md) — working/episodic/long-term memory, the boundaries between them, and the consistency and privacy questions each raises.
4. [RAG and Knowledge Stores as Architecture](04-rag-and-knowledge-stores.md) — vector vs. graph vs. hybrid, retrieval topology, freshness, and retrieval evaluation.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-engineering-curriculum/agentic-systems-architect-solutions).

- [exercise-01: Context budget and rot analysis](exercises/exercise-01-context-budget-and-rot-analysis.md) — instrument a long-horizon run, build a token budget, and locate the rot.
- [exercise-02: Memory-tier placement design](exercises/exercise-02-memory-tier-placement-design.md) — place every piece of agent state into a tier and defend the persistence boundaries.
- [exercise-03: Compaction and note-taking strategy](exercises/exercise-03-compaction-and-notetaking-strategy.md) — design a compaction trigger and a structured-note schema, then prove they preserve task-critical state.
- [exercise-04: RAG architecture for agents](exercises/exercise-04-rag-architecture-for-agents.md) — choose stores, draw the retrieval topology, and specify the retrieval-eval harness.

## Quiz

- [Quiz 1 — Memory & Context Architecture](quizzes/quiz-01-memory-context-architecture.md) — a knowledge check across all four chapters.

## Prerequisites

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — workflow-vs-agent boundaries and the orchestration patterns.
- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — sub-agent isolation is a context-architecture lever as much as an orchestration one.
- Familiarity with retrieval and embeddings at the ai-engineer level (the engineer-track `RAG & Memory` module). This module assumes you can *build* RAG; here you decide *whether and how* it belongs in the architecture.

See [resources.md](resources.md) for primary references.
