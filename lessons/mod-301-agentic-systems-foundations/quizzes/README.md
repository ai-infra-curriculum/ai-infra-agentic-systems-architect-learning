# Quiz — Agentic Systems Foundations & When NOT to Use an Agent

Knowledge check for [mod-301-agentic-systems-foundations](../README.md). Ten questions covering the workflow-vs-agent decision, the orchestration patterns, the tool/sub-agent/hardcoded boundaries, and the start-simple discipline. Answer first, then check yourself against the key at the bottom.

These questions test **judgment**, not recall. Several have a tempting wrong answer that sounds more sophisticated.

## Questions

### 1. Workflow vs. agent

What single property most fundamentally distinguishes a workflow from an agent?

- A. A workflow uses fewer LLM calls than an agent.
- B. A workflow has fixed, code-defined control flow; an agent lets the model direct control flow at runtime.
- C. A workflow cannot use tools; an agent can.
- D. An agent is more accurate than a workflow.

### 2. The bias

Two architects disagree on a borderline system. Per this module's guidance, where should the tie-break land, and why?

- A. Toward the agent, because it handles more cases.
- B. Toward the workflow, because predictability, debuggability, and cost are the defaults you give up only when forced.
- C. Toward whichever is faster to build.
- D. There is no default; it is purely case-by-case.

### 3. Pattern identification

A system translates a document, then summarizes it, then formats the result, with a validation check between steps. Which pattern is this?

- A. Orchestrator-workers
- B. Evaluator-optimizer
- C. Prompt chaining
- D. Routing

### 4. Orchestrator-workers vs. parallelization

What distinguishes orchestrator-workers from parallelization (sectioning)?

- A. Orchestrator-workers run sequentially; parallelization runs concurrently.
- B. In parallelization the subtasks are predetermined; in orchestrator-workers the lead model decides the subtasks at runtime.
- C. Parallelization uses sub-agents; orchestrator-workers uses tools.
- D. They are the same pattern with different names.

### 5. The dangerous pattern

Which orchestration pattern most requires a hard cap to avoid runaway cost, and what cap?

- A. Routing — a cap on the number of categories.
- B. Prompt chaining — a cap on prompt length.
- C. Evaluator-optimizer — a cap on iterations.
- D. Parallelization — a cap on aggregation steps.

### 6. Boundary choice

A capability does a single bounded lookup and returns a small result the model needs in its current reasoning. Which boundary is correct?

- A. Hardcoded path
- B. Tool
- C. Sub-agent
- D. A separate workflow

### 7. The over-engineering anti-pattern

Per Chapter 3, what is the single most common over-engineering mistake at the boundary level?

- A. Hardcoding something that needed model judgment.
- B. Using a sub-agent where a tool call would have done the job.
- C. Using one giant tool instead of several.
- D. Adding too many validation gates to a chain.

### 8. When a sub-agent is justified

Which of these does **not** by itself justify a separate sub-agent?

- A. The sub-task must read a large body of material that should not pollute the parent context.
- B. The sub-task needs a different system prompt and tool set.
- C. The task fans out into independent sub-problems each needing their own reasoning loop.
- D. The capability is called frequently.

### 9. The upgrade gate

A team wants to upgrade from a single LLM call to an autonomous agent "because it could handle edge cases later." Per Chapter 4, what is wrong with this?

- A. Nothing — anticipating edge cases is good architecture.
- B. There is no measured failure at the current rung; "it could" is not evidence, and climbing without evidence violates the gate.
- C. Autonomous agents cannot handle edge cases.
- D. The team should jump straight to orchestrator-workers instead.

### 10. The autonomy ladder

Per this module, where does most production value tend to live on the autonomy ladder?

- A. Rung 5 (autonomous agents), because they are the most capable.
- B. Rungs 1–3 (single calls and simple workflows), which are cheaper, more predictable, and sufficient for most tasks.
- C. Rung 4 (orchestrator-workers), as a universal default.
- D. It is evenly distributed across all rungs.

## Answer key

1. **B** — control flow ownership (code vs. model) is the defining distinction, not call count, tool use, or accuracy. ([Chapter 1](../01-workflows-vs-agents.md))
2. **B** — bias toward the workflow; you give up predictability, debuggability, and cost only when the problem forces it. ([Chapter 1](../01-workflows-vs-agents.md), [Chapter 4](../04-start-simple-discipline.md))
3. **C** — a fixed sequence of LLM calls with gates between them is prompt chaining. ([Chapter 2](../02-orchestration-patterns.md))
4. **B** — the decisive difference is whether subtasks are predetermined (parallelization) or decided dynamically by the lead model (orchestrator-workers). ([Chapter 2](../02-orchestration-patterns.md))
5. **C** — evaluator-optimizer loops and needs a hard iteration cap or it can run indefinitely. ([Chapter 2](../02-orchestration-patterns.md))
6. **B** — a bounded capability returning a small inline result is a tool, not a sub-agent. ([Chapter 3](../03-tool-subagent-boundaries.md))
7. **B** — sub-agent-where-a-tool-would-do is the most common over-engineering: a whole extra loop for work a tool call returns inline. ([Chapter 3](../03-tool-subagent-boundaries.md))
8. **D** — call frequency does not justify a sub-agent; isolation, specialization, or parallel decomposition do. (A frequently called capability is usually a tool or hardcoded path.) ([Chapter 3](../03-tool-subagent-boundaries.md))
9. **B** — the upgrade gate requires a named, measured failure at the current rung; "it could need it" is a YAGNI violation. ([Chapter 4](../04-start-simple-discipline.md))
10. **B** — rungs 1–3 carry most production value; you climb only on evidence. ([Chapter 4](../04-start-simple-discipline.md))
