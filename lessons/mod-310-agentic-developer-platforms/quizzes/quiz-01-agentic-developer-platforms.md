# Quiz 1 — Agentic Developer Platforms & SDLC Integration

Knowledge check for [mod-310](../README.md). Answers are at the bottom; try each question before scrolling. Covers all five chapters.

## Questions

### 1. Extension point: guarantee vs. suggestion

You want the linter to run on **every** file write, no matter what the agent decides. Which extension point?

- A. A skill that describes when to lint.
- B. A `PostToolUse` hook.
- C. An MCP tool the model calls.
- D. A subagent that lints.

### 2. Extension point: external access

The agent needs to read and transition Jira tickets in your internal Jira. Which extension point carries this?

- A. A hook.
- B. A skill.
- C. An MCP server.
- D. The base system prompt.

### 3. The overlap trap

"Reach the internal deployment service" tempts a junior engineer to write it as a *skill*. Why is that wrong?

- A. Skills can't contain instructions.
- B. A skill is procedural know-how; *access* to an external system belongs in an MCP tool. The skill may describe how to deploy, but the MCP server is how the agent calls deploy.
- C. Deployment can't be done by an agent at all.
- D. Skills are deprecated in favor of MCP.

### 4. Why a plugin

What does packaging agents, skills, hooks, and MCP configs into one **plugin** buy a 50-engineer org?

- A. Faster model inference.
- B. A single versioned, installable source of truth, so config stops drifting across engineers' machines.
- C. It bypasses the need for any MCP server.
- D. It makes the agent run locally instead of in the cloud.

### 5. The core trust asymmetry

What is the security architect's foundational assumption when an agent joins the SDLC?

- A. The model is trustworthy; only the tools are risky.
- B. The agent's inputs (tickets, files, comments) are untrusted and its tools are privileged — so the prompt itself can be adversarial.
- C. Inputs are safe once they're inside the org perimeter.
- D. Prompt injection is theoretical and not worth designing against.

### 6. Where credentials live

In a correctly architected platform, where does the Jira token live?

- A. In the agent's system prompt so it can authenticate.
- B. In the conversation context, fetched once per session.
- C. In the MCP server / integration layer — never in the model's context. The agent calls a tool; the tool holds the secret.
- D. Hard-coded in the skill file.

### 7. Sorting actions by gate

Why do production deploys and schema migrations go through a deterministic human gate while reads don't?

- A. Reads are slower.
- B. Reversibility: read freely and write to sandboxed targets freely, but gate durable/irreversible writes — the model proposes, a human or policy disposes.
- C. The model can't perform writes at all.
- D. Deploys cost more tokens.

### 8. Context hydration default

Why is "load the whole repo into context for safety" a bad default?

- A. It violates the MCP spec.
- B. It's slower, costlier, *and less accurate* — relevant signal drowns in noise (context rot).
- C. The context window has no size limit, so it's merely wasteful.
- D. It's actually the recommended default.

### 9. The hydration discipline

What's the one-line discipline for context hydration across all layers?

- A. Pre-load everything once, then cache.
- B. Pull, don't pre-load; distill, don't dump — every token is there because the task needs it now.
- C. Always use a subagent for every read.
- D. Never use MCP tools for context.

### 10. Interface selection

You need a PR plus its checks, its reviews, and its linked issue's status in **one round-trip**. Which interface?

- A. A loop of REST calls.
- B. A GraphQL query shaped to exactly those fields.
- C. A webhook subscription.
- D. Polling the REST API every 5 seconds.

### 11. The polling anti-pattern

The platform needs to act when CI finishes. What's the right design, and what's the anti-pattern?

- A. Right: poll REST in a loop. Anti-pattern: webhooks.
- B. Right: subscribe to a CI completion **webhook/event**. Anti-pattern: polling REST to ask "is it done yet?"
- C. Right: GraphQL subscription only. Anti-pattern: REST writes.
- D. Both are equivalent.

### 12. Webhook robustness

Your webhook handler runs the full 5-minute agent inline before returning `200`. What two things break?

- A. Nothing breaks.
- B. The sender times out and retries (duplicate runs), and you've coupled the slow agent to the fast HTTP handshake — enqueue and return fast instead, and deduplicate on delivery ID.
- C. GraphQL stops working.
- D. The model's context window overflows.

### 13. Adoption timing

Where is adoption of a developer platform most often won or lost?

- A. In the pricing.
- B. In the first ten minutes (activation) and the first failure (trust).
- C. Only in the long-term feature roadmap.
- D. In the choice of programming language.

### 14. Docs that pay twice

Why are skills described as "documentation that pays twice"?

- A. They're written in two languages.
- B. A skill is both human-readable convention *and* the thing the agent loads and applies — so investing in clear skills serves people and enforces behavior at once.
- C. They cost double to maintain.
- D. They're versioned twice.

### 15. Closing the feedback loop

What should happen to every reported bad run on the platform?

- A. It's logged and forgotten.
- B. It becomes a permanent regression **eval case**, so the same failure can't silently return after the next model or API change — and platform changes are gated on the eval suite.
- C. It's escalated to the model provider.
- D. It triggers an immediate model swap.

## Answer key

1. **B** — A guarantee that must run on every event, deterministically, is a hook — not a model-discretionary skill ([Chapter 1](../01-plugin-architecture-extension-standards.md)).
2. **C** — Access to an external system is an MCP server; skills/hooks/prompts don't carry system access.
3. **B** — Procedural know-how (skill) and system access (MCP) are different layers; "reach the service" is access → MCP.
4. **B** — A plugin is one versioned, installable source of truth that collapses per-engineer config drift.
5. **B** — Untrusted inputs + privileged tools; architect assuming the prompt can be adversarial ([Chapter 2](../02-ai-for-sdlc-trust-boundaries.md)).
6. **C** — The credential lives in the integration layer and never enters the model's context.
7. **B** — Sort by reversibility; gate durable/irreversible writes, the model proposes and a human/policy disposes.
8. **B** — Maximal context is slower, costlier, and less accurate due to noise (context rot) ([Chapter 3](../03-tool-use-and-context-hydration.md)).
9. **B** — Pull, don't pre-load; distill, don't dump.
10. **B** — One GraphQL query for deep/related data beats an N+1 REST cascade ([Chapter 4](../04-devops-pm-toolchain-integration.md)).
11. **B** — Subscribe to the completion event; polling REST to detect change is the anti-pattern.
12. **B** — Inline handling causes sender-retry duplicates and couples slow work to the handshake; enqueue, return fast, dedup on delivery ID.
13. **B** — Adoption turns on the first ten minutes and the first failure ([Chapter 5](../05-developer-experience-and-adoption.md)).
14. **B** — Skills are human-readable convention and agent-applied behavior at once — they pay twice.
15. **B** — Every bad run becomes a regression eval case, and platform changes are eval-gated so regressions are caught before engineers hit them.
</content>
