# mod-307-cost-latency-architecture: Cost & Latency Architecture

**Estimated effort:** 10 hours

A multi-agent system that works in a demo and bankrupts you in production is not a working system — it is an unpriced one. By the time an architecture reaches the systems-architect's desk, the question is rarely "can the agents do the task?" It is "what does each task *cost*, how long does it *take*, and is that price defensible against the value it returns?" Token economics, caching, routing, and batching are not optimizations you bolt on after launch. They are load-bearing architectural decisions you make *before* the first diagram is approved — because they determine whether the design is viable at scale at all.

This module makes cost and latency first-class architectural concerns. You will learn to model token economics for fan-out systems where one user request becomes forty model calls, set explicit value thresholds that decide whether a task even deserves an agent, wield prompt caching / model routing / batch processing as deliberate levers with known multipliers, and bake cost models and SLAs directly into a reference architecture. The deliverable mindset is the architect's: a cost model you can defend to a CFO and a budget you can defend to an SRE.

> **Price the architecture, not the prompt.** Engineers optimize a call. Architects decide which calls happen, on which model, against which cache, in which lane — and put a number on each before anyone writes the code. A design without a cost model is a guess.

## Learning objectives

- Model **token economics** for multi-agent systems — per-task and per-tenant — and set explicit **value thresholds** that decide agentic vs. workflow designs.
- Apply **prompt/prefix caching, model routing, and batch processing** as architectural levers with known cost and latency multipliers, not ad-hoc tweaks.
- Build **cost models and SLAs** into a reference architecture so unit economics and latency targets are visible at design time.
- Trade off **latency, cost, and quality** under explicit budgets, and defend where you spend and where you cut.

## Lecture chapters

1. [Token Economics & Value Thresholds](01-token-economics.md) — how tokens become dollars in fan-out systems, the unit-economics model, and the threshold test for agent vs. workflow.
2. [Architectural Levers: Caching, Routing, and Batching](02-caching-routing-batching.md) — prefix caching, model routing, and the Batches API as levers with quantified multipliers.
3. [Cost Models & SLAs in the Reference Architecture](03-cost-models-and-slas.md) — making unit economics and latency SLAs load-bearing parts of the design, with guardrails and kill switches.
4. [The Cost–Latency–Quality Triangle](04-cost-latency-quality-budgets.md) — explicit budgets, the three-way trade-off, and defending where you spend.

## Exercises

Hands-on design practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-engineering-curriculum/agentic-systems-architect-solutions).

- [exercise-01: Token economics cost model](exercises/exercise-01-token-economics-cost-model.md) — build a per-task and per-tenant cost model for a fan-out system and derive its value threshold.
- [exercise-02: Caching and routing architecture](exercises/exercise-02-caching-and-routing-architecture.md) — design the caching and routing layers, quantify the multipliers, and diagram the routed architecture.
- [exercise-03: Cost–latency–quality budgets](exercises/exercise-03-cost-latency-quality-budgets.md) — set explicit budgets, resolve the three-way trade-off, and write the SLA and ADRs.

## Prerequisites

- [mod-301: Agentic Systems Foundations](../mod-301-agentic-systems-foundations/README.md) — the workflow-vs-agent decision and the orchestration pattern catalog. This module puts a price on those choices.
- [mod-302: Multi-Agent Orchestration](../mod-302-multi-agent-orchestration/README.md) — orchestrator-worker fan-out is the cost driver this module models.
- Comfort reading architecture diagrams, building a spreadsheet/worksheet model, and writing architecture decision records (ADRs).
- See [PREREQUISITES.md](../../PREREQUISITES.md) for the role-level entry skills.

See [resources.md](resources.md) for primary references. Anthropic's prompt-caching, pricing, and Message Batches documentation are the anchor texts for the architectural levers in this module.
