# mod-301-agentic-systems-foundations: Agentic Systems Foundations & When NOT to Use an Agent

**Estimated effort:** 12 hours

Most agentic systems that fail in production did not fail because the model was weak. They failed because someone reached for an agent where a workflow would have been cheaper, more predictable, and easier to debug — or because they hardcoded a path that should have been left to the model. The architect's first job is not to build the agent. It is to decide *whether* there should be an agent at all, and where in the system the model's judgment is allowed to operate versus where the path is fixed in code.

This module establishes the design vocabulary and the decision discipline for everything that follows in the role. You will not implement the reason-act loop here — that is engineering work covered upstream. You will learn to *choose* the architecture: when a deterministic workflow beats a model-driven agent, which of Anthropic's orchestration patterns fits a given task, where the tool/sub-agent/hardcoded-path boundaries fall, and how to defend "start simple" against the gravitational pull toward unnecessary autonomy.

> **Design over implementation.** Your deliverables in this module are decision matrices, reference-architecture diagrams, and architecture decision records (ADRs) — not running code. The judgment you build here is what separates an architect from an engineer who happens to draw boxes.

## Learning objectives

- Distinguish **workflows from agents** and justify the choice per task, trading predictability against model-driven flexibility.
- Apply Anthropic's **orchestration patterns** — prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer — and recognize which task shape each one fits.
- Decompose a system into **tool vs. sub-agent vs. hardcoded-path** boundaries, and defend where each boundary falls.
- Frame the **"start simple, add complexity only when it demonstrably improves outcomes"** discipline as an explicit, auditable design gate.

## Lecture chapters

1. [Workflows vs. Agents: The Foundational Choice](01-workflows-vs-agents.md) — predictability vs. flexibility, the cost of autonomy, and a decision test you can defend.
2. [The Orchestration Pattern Catalog](02-orchestration-patterns.md) — prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer, and the task shape each one fits.
3. [Drawing the Boundaries: Tool, Sub-Agent, or Hardcoded Path](03-tool-subagent-boundaries.md) — what belongs in code, what belongs in a tool, and what justifies a separate context window.
4. [The Discipline of Starting Simple](04-start-simple-discipline.md) — complexity as a cost you must justify, the autonomy ladder, and the upgrade gate.

## Exercises

Hands-on design practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

- [exercise-01: Workflows vs. agents decision matrix](exercises/exercise-01-workflows-vs-agents-decision-matrix.md) — build and defend a reusable decision matrix across five candidate systems.
- [exercise-02: Orchestration pattern catalog](exercises/exercise-02-orchestration-pattern-catalog.md) — map real tasks to the five patterns and document the tradeoffs.
- [exercise-03: Tool vs. sub-agent decomposition](exercises/exercise-03-tool-vs-subagent-decomposition.md) — draw the boundary lines for a support-automation system and record the decisions as ADRs.
- [exercise-04: Reference architecture teardown](exercises/exercise-04-reference-architecture-teardown.md) — reverse-engineer a published agentic system and critique its design choices.

## Prerequisites

- Working familiarity with how an agent loop and tool calling operate (the reason-act loop, structured tool calls). You do not implement them here, but you must be able to read them.
- Comfort reading architecture diagrams and writing architecture decision records (ADRs).
- See [PREREQUISITES.md](../../PREREQUISITES.md) for the role-level entry skills.

See [resources.md](resources.md) for primary references. Anthropic's *Building effective agents* is the anchor text for this entire module.
