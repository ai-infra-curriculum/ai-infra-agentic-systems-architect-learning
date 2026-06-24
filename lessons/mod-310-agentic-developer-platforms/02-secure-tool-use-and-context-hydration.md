# Chapter 2 — Secure Tool-Use and Context Hydration

The moment an agent acts on internal data — your source, your tickets, your customer records, your production systems — two design problems become the architect's responsibility, and they are intertwined. **Tool-use** is how the agent *acts*: which functions it can call, on which targets, with what privilege. **Context hydration** is what the agent *knows*: which slices of internal data are pulled into its window for a given task. Get tool-use wrong and the agent can take an action it should never be able to take. Get hydration wrong and it either acts blind (too little context) or drowns in noise and leaks data (too much). This chapter designs both, around a single uncomfortable fact about agentic systems.

## The trust asymmetry: untrusted input, privileged tools

Here is the assumption every secure agent platform is built on:

> **The agent's inputs are untrusted, and its tools are privileged.** A ticket, a file, a code comment, a support message, a row in a database — any of these can carry instructions an attacker planted. The agent reads them into the same context that decides which privileged tool to call next. So the prompt itself can be adversarial.

This is *prompt injection*, and at platform scale it is not theoretical. An agent that reads internal data and can also write to internal systems is exactly the configuration the threat targets: untrusted text flows in, a privileged action flows out, and nothing in between is inherently a boundary. The architect's job is to *put boundaries there on purpose* — because the model will not reliably draw them for you.

```text
   UNTRUSTED                                         PRIVILEGED
   ─────────                                         ──────────
   ticket text ─┐                                  ┌─▶ deploy
   file content ├─▶  agent context (reasoning)  ──▶┤   write DB
   PR comment ──┤      ▲ attacker-controllable      ├─▶ send message
   DB row ──────┘      │ text reaches here          └─▶ external API
                       │
            ── the model is NOT a trust boundary ──
            ── you must place real boundaries on the tool layer ──
```

The defenses live on the **tool layer and the integration layer**, not in a prompt asking the model to behave.

## Scoped, least-privilege tool design

A tool is a privilege grant. Designing the tool *is* designing the blast radius. Four disciplines:

**1. Scope the target, not just the verb.** "Write a file" is too broad; "write a file *under the workspace, not outside it*" is a scoped tool. "Run a query" is too broad; "run a *read-only* query against the *reporting replica*" is scoped. Bake the constraint into the tool's implementation so the model cannot widen it by asking nicely.

**2. Sort actions by reversibility and gate accordingly.** Not all tool calls deserve the same trust.

```text
   reversibility ladder
   ─────────────────────────────────────────────────────────
   READ (internal data)          → allow freely, scoped to need
   WRITE to sandbox/branch/draft → allow freely (cheap to undo)
   WRITE durable / irreversible  → GATE  (human or policy decides)
     (deploy, prod schema change, payment, customer email,
      ticket close, account creation)
```

The model **proposes**; a human or a policy **disposes** on the irreversible tier. This is the same human-in-the-loop discipline the deployment module designs as a durable gate — here it is the security control that keeps an injected prompt from reaching an irreversible action unsupervised.

**3. Keep credentials out of the model's context.** The token for the internal API lives in the **integration/MCP layer**, never in the system prompt and never in the conversation. The agent calls a tool by name; the tool holds the secret and attaches it. If a credential ever enters the context window, it can be exfiltrated by an injected instruction. The boundary is hard: the model knows *that* it can call `deploy`, never *how* `deploy` authenticates.

**4. Make tools observable and revocable.** Every privileged call emits an audit record (who/which run, what target, what arguments, what result), and the platform can revoke a tool or narrow its scope centrally — through the packaged bundle of [Chapter 1](01-extension-model-across-platforms.md) — without touching every agent.

## Context hydration: pull, don't pre-load

Now the other half. The agent needs *context* — relevant internal data — to do real work. The naive instinct is "load everything so it can't miss anything": dump the whole repo, the whole ticket history, the whole knowledge base into the window. This is wrong on three axes at once:

- **Cost** — every token is paid for, every turn.
- **Latency** — bigger context is slower to process.
- **Accuracy** — and this is the one people underrate: **more context is often *less* accurate.** Relevant signal gets buried in irrelevant noise, and model attention degrades over a bloated window. This is *context rot*: the failure mode where adding context makes the agent dumber, not smarter.

The discipline, stated once for every layer:

> **Pull, don't pre-load; distill, don't dump.** Every token in the window is there because *this task, right now* needs it — fetched on demand and compressed to its essence, not pre-loaded "just in case."

### Layered hydration

Context arrives in layers, each with a different job and a different budget:

```text
   ┌─────────────────────────────────────────────────────────┐
   │ L1  STANDING context  (small, always present)            │
   │     org conventions, the agent's role, safety rules       │
   ├─────────────────────────────────────────────────────────┤
   │ L2  TASK context  (fetched per task, scoped to the goal)  │
   │     the specific ticket, the specific file(s), the spec   │
   ├─────────────────────────────────────────────────────────┤
   │ L3  RETRIEVED context  (pulled on demand, distilled)      │
   │     search results, related docs, related code —          │
   │     summarized to the relevant slice, not pasted whole     │
   ├─────────────────────────────────────────────────────────┤
   │ L4  TOOL-RESULT context  (just-returned, pruned after use)│
   │     output of the last tool call; drop once consumed       │
   └─────────────────────────────────────────────────────────┘
```

- **L1 standing context** is tiny and stable: who the agent is, the org's conventions, the non-negotiable safety rules. Loaded once.
- **L2 task context** is the specific thing this task is about — *this* ticket, *these* files, *this* spec — fetched per task and scoped to the goal, never "the whole project."
- **L3 retrieved context** is the on-demand layer: the agent searches, retrieves, and **distills** related material to the relevant slice. A 40-page design doc becomes the three paragraphs that bear on the task, not 40 pasted pages.
- **L4 tool-result context** is the transient output of the last tool call, pruned once consumed so it does not accumulate.

The budget discipline: each layer has a ceiling, and L2–L4 are *fetched* and *pruned*, not retained. A platform that pre-loads L2/L3 "to be safe" will be slower, costlier, *and* less accurate than one that hydrates just-in-time.

## Degrading safely when context is missing

Real internal systems are flaky. The knowledge base is down, the ticket has no description, the relevant file was deleted, the search returns nothing. A platform-grade agent must **degrade safely**, not confidently hallucinate over the gap.

```text
   context fetch ──▶ success ──▶ proceed with hydrated context
        │
        └─▶ missing / partial / stale
                 │
                 ├─▶ state the gap explicitly ("no spec found for TICKET-123")
                 ├─▶ narrow the action to what the partial context supports
                 ├─▶ escalate to a human gate if the missing context guarded
                 │     an irreversible action
                 └─▶ NEVER fabricate the missing context to keep going
```

The design rule: **missing context narrows the action; it never gets invented.** If the spec that would justify a risky change is missing, the agent does the safe subset and flags the gap — it does not imagine a spec. And missing context that guarded an irreversible action *promotes* that action up the reversibility ladder into a human gate. Safe degradation is an explicit branch you design, not an accident you hope for.

## Worked judgment

*"Give the support agent access to the customer database so it can answer questions."* — Decompose into tool-use and hydration. **Tool-use:** not "access the database" (too broad) but a *read-only, scoped* tool — `lookup_customer(id)` returning a fixed, minimal field set from the reporting replica, with the credential in the integration layer and every call audited. Any *write* (refund, account change) is on the irreversible tier and **gated**. **Hydration:** L2 pulls *this* customer's relevant fields per query; it does not pre-load the customer table. **Degradation:** customer not found → say so and stop, never fabricate a record.

*"The code agent keeps missing context, so let's just feed it the whole repo."* — This is the context-rot trap. The fix is not more context; it is *better-targeted* context: L2 = the specific files the ticket touches; L3 = distilled search over the rest, pulled on demand. The whole-repo dump is slower, costlier, and *less* accurate — and it also widens the data exposure surface for an injected prompt.

## Key takeaways

- The foundational assumption: **inputs are untrusted, tools are privileged, and the model is not a trust boundary.** Place real boundaries on the tool and integration layers — prompt injection at platform scale is a design problem, not a behavior request.
- Design tools as **scoped, least-privilege** grants: constrain the target not just the verb, **sort actions by reversibility and gate the irreversible tier**, keep **credentials out of the model's context**, and make every privileged call observable and revocable.
- Context hydration discipline: **pull, don't pre-load; distill, don't dump.** Hydrate in layers (standing / task / retrieved / tool-result), budget each, and fetch-and-prune L2–L4 rather than retaining them — because **more context is often less accurate** (context rot).
- **Degrade safely**: missing context narrows the action and is stated explicitly; it is never fabricated, and missing context guarding an irreversible action promotes that action to a human gate.
