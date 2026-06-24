# Chapter 3 — Drawing the Boundaries: Tool, Sub-Agent, or Hardcoded Path

Once you have chosen a pattern (Chapter 2), the next architectural decision is granular: for each capability in the system, where does it live? There are three homes, and putting a capability in the wrong one is a quiet, recurring source of cost and unreliability.

- **Hardcoded path** — deterministic code. No model decides whether or how it runs; the engineer wrote the steps.
- **Tool** — a function the model may call within a single context. The model decides *whether* and *with what arguments*, but the work itself is code, and the result returns to the same context.
- **Sub-agent** — a separate agent loop with its **own context window**, invoked to do a bounded chunk of work and return a **distilled** result.

The decision is not stylistic. Each boundary has a different cost profile and a different failure mode.

## The decision tree

```text
 Does this capability need a model decision at all?
        │
        ├── No ──▶ HARDCODED PATH
        │          (deterministic logic, validation, routing you already know)
        │
        └── Yes ─▶ Does it need its own context window / specialization /
                   isolation from the main context?
                        │
                        ├── No ──▶ TOOL
                        │          (model decides whether to call; result returns inline)
                        │
                        └── Yes ─▶ SUB-AGENT
                                   (separate loop, distilled return)
```

## When to hardcode

Hardcode anything where **the model adds no judgment** and a fixed path is more reliable:

- Input validation, schema checks, auth, and rate limits.
- Routing you can already enumerate (a known set of categories → known handlers).
- Sequencing that is always the same (always fetch context *before* generating; always log *after* acting).
- Anything regulated or safety-critical where you need deterministic, auditable behavior.

Hardcoding is not a failure of ambition. Every step you can move out of the model's hands is a step you can test, replay, and audit. The architect's instinct should be: **hardcode by default, escalate to model judgment only where judgment is actually required.**

## When to use a tool

Make it a tool when the model genuinely needs to **decide whether and how** to use a capability, but the work is bounded and its result belongs in the current line of reasoning:

- The decision to call depends on context the model holds ("the user asked about an order, so look up the order").
- The result is compact enough to return inline without flooding the context.
- The work does not need its own reasoning loop — it is a single function call, not a sub-problem.

A tool is the right boundary for *capabilities*: search, lookup, calculate, send. Tool design quality (clear names, typed schemas, good descriptions, error messages the model can recover from) is itself an architectural concern — a poorly specified tool produces wrong calls no model can fix.

## When to use a sub-agent

Spin up a sub-agent only when the work justifies a **separate context window**. Three things justify it:

1. **Context isolation.** The sub-task involves reading a lot of material (twenty documents, a large codebase) whose raw content must not pollute the main context. The sub-agent reads it all; the parent sees only the distilled result.
2. **Specialization.** The sub-task wants a different system prompt and a different tool set (a `security-reviewer` sub-agent with scanning tools and a security-focused prompt).
3. **Parallel decomposition.** The task fans out into independent sub-problems that each need their own reasoning loop (orchestrator-workers from Chapter 2).

The economics: a sub-agent costs an entire extra agent loop, but it *protects the parent's context budget*. That trade is worth it only when the isolation or specialization is real. Spinning up a sub-agent to do what a tool call could have done is pure overhead — extra latency, extra tokens, and another trajectory that can fail.

## Cost and failure-mode comparison

| Boundary | Who decides | Context cost | Failure mode | Reach for it when |
| --- | --- | --- | --- | --- |
| Hardcoded path | Engineer (no model) | None | Bug in code (reproducible) | No model judgment needed |
| Tool | Model: whether/how to call | Result returns inline | Wrong call, or context bloat from large results | Bounded capability, compact result |
| Sub-agent | Parent model: whether to spawn | Separate window; only distilled result returns | Bad decomposition, distillation loses detail | Isolation, specialization, or parallel decomposition |

## The anti-patterns

- **Sub-agent where a tool would do.** The single most common over-engineering. If the work is one function call returning a small result, it is a tool. A sub-agent there just adds a loop that can fail.
- **Tool where a hardcoded path would do.** Letting the model "decide" something that is always the same. You have added a non-deterministic decision to a deterministic process — cost and variance for nothing.
- **Hardcoded path where judgment is required.** The opposite failure: encoding a brittle if/else ladder for something the input genuinely varies on, then watching it break on the case you did not foresee. This is the case that *should* move right.
- **Everything in one giant tool.** A "do_everything" tool with a mode flag. Split it; the model routes better to well-named, single-purpose tools.

## Worked decomposition

A support-automation system, capability by capability:

- *Authenticate the user* → **hardcoded** (no judgment, security-critical).
- *Classify the ticket category* → **tool** or a routing call (model judgment, compact result). If categories are fixed and the classifier is cheap, this is just routing.
- *Look up the order* → **tool** (model decides whether it is needed; result is small).
- *Research a thorny multi-system issue across logs and docs* → **sub-agent** (large context, distilled return).
- *Send the resolution email* → **hardcoded** trigger after a model-drafted body (the *decision* to send may be gated by a human; the *act* of sending is code).

Notice the pattern: the model's judgment is fenced into the few places it is actually needed, and surrounded by deterministic code.

## Key takeaways

- Three homes for every capability: **hardcoded path**, **tool**, **sub-agent** — distinguished by who decides and what context it costs.
- **Hardcode by default**; escalate to a tool only where the model must decide whether/how, and to a sub-agent only where a separate context window is genuinely justified.
- The dominant anti-pattern is **sub-agent-where-a-tool-would-do**: a whole extra loop for work a single tool call could return inline.
- Fence model judgment into the few places it is required and surround it with deterministic code you can test, replay, and audit.
