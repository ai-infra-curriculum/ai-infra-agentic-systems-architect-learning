# Quiz 1 — Agentic Developer & Internal Platforms

Knowledge check for [mod-310](../README.md). Answers are at the bottom; try each question before scrolling. Covers all five chapters. No question assumes a single host platform or a single (SDLC) context — the architecture is generic.

## Questions

### 1. The generic extension model

The four generic extension points under every host's branding are:

- A. Agents, prompts, files, and APIs.
- B. Agents, skills/tools, hooks/lifecycle events, and MCP (external access).
- C. Models, datasets, pipelines, and dashboards.
- D. REST, GraphQL, webhooks, and polling.

### 2. Choosing the extension point

You want a security linter to run on **every** artifact the agent produces, no matter what the model decides. Which generic extension point?

- A. A skill that describes when to lint.
- B. A hook on a lifecycle event (deterministic, runs every time).
- C. An MCP tool the model may choose to call.
- D. A subagent that lints.

### 3. The misplacement trap

"Reach the internal deploy service" tempts an engineer to write it as a *skill*. Why is that wrong?

- A. Skills can't contain instructions.
- B. A skill is procedural know-how; *access* to an external system belongs in MCP. A skill may describe *when* to deploy, but the MCP server is *how* the agent calls deploy.
- C. Deployment can't be done by an agent at all.
- D. Skills are deprecated.

### 4. Platforms as peers

Across Claude Code, Cursor, GitHub Copilot, LangGraph Platform, and a custom host, which row of the extension model **diverges the most** between hosts?

- A. The MCP / external-access row.
- B. The agents row.
- C. The hooks / lifecycle-events row — some hosts have rich deterministic interception, others are thin.
- D. The tools row.

### 5. The portability anchor

Which part of an agent platform's architecture is the **most portable** across hosts, and why?

- A. Hooks, because every host implements them identically.
- B. MCP servers, because MCP is an open protocol — the same server rebinds to any MCP-capable host.
- C. The host's packaging manifest, because it's standardized.
- D. The system prompt, because prompts never change.

### 6. Grading lock-in

A platform whose core guarantees live entirely in one vendor's proprietary lifecycle/hook model is best described as:

- A. Cheap to re-host.
- B. Fully portable.
- C. Expensive to leave — which may be an acceptable trade, but only if it was chosen, not stumbled into.
- D. Impossible to build.

### 7. The trust asymmetry

What is the foundational security assumption when an agent acts on internal data?

- A. The model is trustworthy; only the tools are risky.
- B. The agent's inputs (tickets, files, logs, customer messages) are untrusted and its tools are privileged — so the prompt itself can be adversarial, and the model is not a trust boundary.
- C. Inputs are safe once inside the org perimeter.
- D. Prompt injection is theoretical.

### 8. Where credentials live

In a correctly architected platform, where does the internal-system token live?

- A. In the agent's system prompt.
- B. In the conversation context, fetched once per session.
- C. In the integration / MCP layer — never in the model's context. The agent calls a tool; the tool holds the secret.
- D. Hard-coded in a skill file.

### 9. Sorting actions by gate

Why do irreversible writes (deploy, prod schema change, refund, customer email) go through a gate while reads don't?

- A. Reads are slower.
- B. Reversibility: read and sandbox-write freely, but gate durable/irreversible writes — the model proposes, a human or policy disposes.
- C. The model can't perform writes at all.
- D. Writes cost more tokens.

### 10. Context hydration default

Why is "load everything into context for safety" a bad default?

- A. It violates the MCP spec.
- B. It's slower, costlier, *and less accurate* — relevant signal drowns in noise (context rot).
- C. The context window has no size limit, so it's merely wasteful.
- D. It's actually recommended.

### 11. The hydration discipline

What is the one-line discipline for context hydration across all layers?

- A. Pre-load everything once, then cache.
- B. Pull, don't pre-load; distill, don't dump — every token is there because the task needs it now.
- C. Always use a subagent for every read.
- D. Never use MCP for context.

### 12. Safe degradation

The knowledge base is down and the spec that would justify a risky change is missing. What does a platform-grade agent do?

- A. Fabricate a plausible spec and proceed.
- B. State the gap, narrow to the safe subset, and promote any guarded irreversible action into a human gate — never invent the missing context.
- C. Crash the run.
- D. Load the entire knowledge base instead.

### 13. Interface selection

You need a work item plus its checks, its reviews, and its linked item's status in **one round-trip**. Which interface?

- A. A loop of REST calls.
- B. A GraphQL query shaped to exactly those fields.
- C. A webhook subscription.
- D. Polling REST every 5 seconds.

### 14. The polling anti-pattern

The platform must act when a pipeline finishes. What's the right design, and the anti-pattern?

- A. Right: poll REST in a loop. Anti-pattern: webhooks.
- B. Right: subscribe to a completion **event/webhook**. Anti-pattern: polling REST to ask "is it done yet?"
- C. Right: GraphQL subscription only. Anti-pattern: REST writes.
- D. Both are equivalent.

### 15. Webhook robustness

Your webhook handler runs the full multi-minute agent inline before returning `200`. What breaks?

- A. Nothing.
- B. The sender times out and retries (duplicate runs), and you've coupled slow work to the fast handshake — enqueue and return fast, and dedupe on delivery ID.
- C. GraphQL stops working.
- D. The context window overflows.

### 16. Adoption timing

Where is adoption of an internal platform most often won or lost?

- A. In pricing.
- B. In the first ten minutes (activation) and the first failure (trust).
- C. Only in the long-term roadmap.
- D. In the programming language.

### 17. Docs that pay twice

Why are skills described as "documentation that pays twice"?

- A. They're written in two languages.
- B. A skill is both human-readable convention *and* the artifact the agent loads and applies — so investing in clear skills serves people and steers the agent at once.
- C. They cost double to maintain.
- D. They're versioned twice.

### 18. Closing the feedback loop

What should happen to every reported bad run?

- A. It's logged and forgotten.
- B. It becomes a permanent regression **eval case**, and every platform change (new model, new API, new skill, widened tool) is gated on the eval suite — so the same failure can't silently return.
- C. It's escalated to the model provider.
- D. It triggers an immediate model swap.

### 19. Generalizing the platform

When you re-target an agent platform from software development to a non-developer domain (ops/support/analytics), what changes and what stays constant?

- A. Everything must be rebuilt from scratch.
- B. The **nouns** change (inputs, tools, external systems, irreversible-action list); the **architecture** stays constant (extension model, trust asymmetry, hydration, interface selection, DX).
- C. The architecture changes but the tools stay the same.
- D. Only the model changes.

### 20. The generalization test

If your platform only works for SDLC and would require a full rebuild for a support or ops domain, what does that tell you?

- A. You architected a reusable platform pattern.
- B. You built an SDLC tool, not a platform — generalization is the test that proves you architected a pattern rather than wired a single tool.
- C. The domains are fundamentally incompatible.
- D. The model is too small.

## Answer key

1. **B** — The four generic extension points are agents, skills/tools, hooks/lifecycle, and MCP for external access ([Chapter 1](../01-extension-model-across-platforms.md)).
2. **B** — A guarantee that must run on every event deterministically is a hook, not a model-discretionary skill.
3. **B** — Procedural know-how (skill) and system access (MCP) are different kinds of capability; "reach the service" is access → MCP.
4. **C** — Hooks/lifecycle is the row where hosts diverge most; MCP, agents, and tools are more uniform table stakes.
5. **B** — MCP is an open protocol, so MCP servers rebind to any MCP-capable host; the external-access layer is the portability anchor.
6. **C** — Guarantees in a proprietary lifecycle make a platform expensive to leave; acceptable only if the trade was chosen, not stumbled into ([Chapter 1](../01-extension-model-across-platforms.md)).
7. **B** — Untrusted inputs + privileged tools; the model is not a trust boundary, so the prompt can be adversarial ([Chapter 2](../02-secure-tool-use-and-context-hydration.md)).
8. **C** — The credential lives in the integration/MCP layer and never enters the model's context.
9. **B** — Sort by reversibility; gate durable/irreversible writes — the model proposes, a human/policy disposes.
10. **B** — Maximal context is slower, costlier, and less accurate due to noise (context rot) ([Chapter 2](../02-secure-tool-use-and-context-hydration.md)).
11. **B** — Pull, don't pre-load; distill, don't dump.
12. **B** — State the gap, narrow to the safe subset, promote guarded irreversible actions to a gate; never fabricate missing context.
13. **B** — One shaped GraphQL query for deep/related data beats an N+1 REST cascade ([Chapter 3](../03-workflow-toolchain-integration.md)).
14. **B** — Subscribe to the completion event; polling REST to detect change is the anti-pattern.
15. **B** — Inline handling causes sender-retry duplicates and couples slow work to the handshake; enqueue, return fast, dedupe on delivery ID.
16. **B** — Adoption turns on the first ten minutes and the first failure ([Chapter 4](../04-platform-developer-experience-and-adoption.md)).
17. **B** — A skill is human-readable convention and agent-applied behavior at once — it pays twice.
18. **B** — Every bad run becomes an eval-gated regression case, so regressions are caught before users hit them.
19. **B** — Re-targeting re-populates the nouns and leaves the architecture constant ([Chapter 5](../05-beyond-sdlc-generalizing-the-platform.md)).
20. **B** — If it only works for SDLC, it's an SDLC tool; generalization proves you built a pattern, not a single wired-up tool.
