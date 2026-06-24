# Chapter 3 — Human-in-the-Loop Approval Architecture

Some agent actions are too consequential to take autonomously: wiring money, deleting production data, emailing a customer, deploying to prod, filing a legal document. For these, the architecture must **stop the agent, surface the proposed action to a human, and resume only on the human's decision**. This is human-in-the-loop (HITL), and as an architect your job is to design the *interrupt* — not as a synchronous "block this thread until someone clicks" hack, but as a **durable, resumable gate** whose state survives an arbitrary wait and a process restart.

The naive version — `result = input("approve? ")` blocking the agent process — fails the moment the wait is longer than the process lives. A human might approve in thirty seconds or thirty hours; in between, a deploy rolls the pod, the node drains, the process dies. If the approval state lived in that process, the run is lost and the human's eventual click lands nowhere. HITL done right is built directly on the durable-execution substrate of [Chapter 1](01-durable-execution-and-resumption.md): the gate is a **durable interrupt** the run sleeps on, and the human's decision is a **signal** that wakes it.

## The four-action decision set: approve / edit / reject / respond

A HITL gate is not a boolean. LangGraph's interrupt model names the four responses a human can give, and they are the right vocabulary because each resumes the run differently:

```text
   agent proposes action A  ──▶  ⟦ HITL GATE ⟧  ──▶ awaits human decision
                                                          │
       ┌───────────────┬──────────────────┬──────────────┴───────────┐
       ▼               ▼                  ▼                          ▼
   APPROVE          EDIT               REJECT                    RESPOND
   run A as-is   run A' (human-     do NOT run A;            don't run A; inject
                 modified args)     return control to        human's message as
                                    agent to replan          tool/observation,
                                                             agent continues
```

- **Approve** — execute the proposed action exactly as the agent specified it. The run resumes past the gate with the action's result.
- **Edit** — execute, but with **human-modified arguments**. The human corrects the recipient, trims the amount, fixes the query; the *edited* action runs, and the run resumes with that result. The agent must treat the edited args as authoritative.
- **Reject** — do **not** execute. Control returns to the agent with a rejection signal so it can replan, choose a different action, or give up. Reject is not "error" — it is a legitimate branch the agent must handle.
- **Respond** — do not execute the action; instead inject the human's free-text **response as an observation** (as if a tool returned it), and let the agent continue reasoning. This is the "answer the agent's question / give it guidance" path.

Designing a HITL gate means specifying, for *that* gate, which of the four are valid and what each does to the run's state. A "release funds" gate might allow approve/edit/reject but not respond; a "should I escalate?" gate might be respond-only. **Enumerating the valid decision set per gate is the core design artifact.**

## The gate must be a durable interrupt

Here is the architecture that makes the wait survivable. The run does not hold a thread open. It records that it is *waiting at gate G for decision D*, persists that, and yields. The worker is now free; the process can die. When the human decides — minutes or days later, possibly on a completely different worker after a deploy — their decision is delivered as a **signal**, the engine wakes the run, and it resumes from the gate.

```text
   ┌────────────┐   1. reach gate, persist {gate:G, proposed:A,            ┌──────────┐
   │  agent run │──────  allowed:[approve,edit,reject], status:WAITING} ──▶│ durable  │
   │ (workflow) │                                                          │  store   │
   └─────┬──────┘   2. YIELD worker (no thread held; process may die)      └────┬─────┘
         │                                                                      │
         ▼                                                                      │
   ── process can be killed / deployed / drained here; state is safe ──         │
         │                                                                      │
   ┌─────┴──────┐   4. SIGNAL(decision) wakes run on ANY worker          ┌──────┴─────┐
   │  resumed   │◀───────────────────────────────────────────────────── │  approval  │
   │  run       │   5. validate decision, apply branch, continue          │   UI/API   │
   └────────────┘      ◀───────── 3. human decides (mins…days) ────────── └────────────┘
```

In Temporal this is `await workflow.wait_condition(...)` on a signal; in LangGraph it is `interrupt()` with a persisted checkpoint resumed by a `Command`. The mechanism differs; the contract is identical and is what you must specify:

- **What is persisted at the gate** — the proposed action, its arguments, the run/gate identity, the allowed decision set, and a deadline.
- **How the decision is delivered** — a signal/command carrying `{decision, edited_args?, response_text?, decider_identity}`, correlated to the run by a stable **run + gate ID** (never by a UI session, which won't survive the wait).
- **Idempotent resume** — the same decision delivered twice (network retry on the approval API) must resume the run once. The gate consumes the *first* valid decision for that gate ID and ignores duplicates, reusing the at-least-once idempotency discipline from [Chapter 1](01-durable-execution-and-resumption.md).

## Authority, audit, and the approval contract

A HITL gate is also a **governance** boundary (this is where mod-308 touches mod-309). The architecture must answer, durably:

- **Who may decide?** The gate carries an **authority policy** — which role/identity is permitted to approve *this class* of action, and at what value thresholds (e.g., "transfers over $50k require two approvers"). The decider's identity is validated on the signal, not trusted from the UI.
- **What is the record?** Every gate records an immutable audit entry: *proposed action, by which agent run, at what time; decision, by whom, when; edited arguments if any.* Because this rides the durable event history, it is tamper-evident and replayable — a direct payoff of building HITL on durable execution rather than bolting it on.
- **What if no one decides?** Every gate needs a **timeout policy** with an explicit default: after the deadline, does the gate auto-reject (safe default for irreversible actions), escalate to a higher authority, or notify and keep waiting? An approval that can wait forever stalls the run *and*, per [Chapter 2](02-inflight-safe-deployment.md), pins an old code version alive indefinitely. The TTL closes both problems.

```text
   approval contract (per gate)
   ─────────────────────────────────────────────
   allowed_decisions : subset of {approve,edit,reject,respond}
   authority_policy  : who may decide; thresholds; multi-approver rules
   timeout           : deadline + default action (reject | escalate | notify)
   audit_record      : proposed / decided / by-whom / when / edits — immutable
   correlation_key   : (run_id, gate_id)  — stable across the wait
   idempotency       : first valid decision wins; duplicates ignored
```

## Where to place gates: a design discipline

Gates are not free — each one introduces human latency (seconds to days) and an operational burden (someone must be on the hook to decide). Place them by **blast radius**, not by habit:

- **Gate the irreversible and the high-consequence.** Money movement, data deletion, external communications, production changes, anything you cannot cleanly undo. These are exactly the side effects you already enumerated for idempotency in [Chapter 1](01-durable-execution-and-resumption.md) — the lists overlap heavily.
- **Do not gate the cheap and reversible.** Reading a document, drafting (not sending) text, internal computation. Gating these just trains humans to rubber-stamp, which destroys the value of the gate.
- **Prefer one gate at the consequential action over many small gates.** A gate per tool call produces approval fatigue; a single gate at "here is the plan / here is the irreversible step" preserves human attention for where it matters.
- **Make the proposal reviewable.** The human can only make a good decision if the gate surfaces *what* will happen and *why* — the proposed action, its arguments, and the agent's rationale. A gate that says "approve?" with no context is a rubber stamp by design.

The architect's deliverable here is a **gate-placement map**: for a given agent, which actions are gated, with which decision set, authority policy, and timeout. That map is reviewed alongside the side-effect/idempotency list, because they answer the same underlying question — *which actions are consequential enough to need special handling?*

## Worked judgment

*"A coding agent that edits files in a sandbox branch."* — Reversible (it is a branch; review happens at PR time). **No inline HITL gate**; the existing code-review process is the human loop. Adding a per-edit approval would create fatigue for zero safety gain.

*"A finance agent that can initiate wire transfers."* — Irreversible, high-consequence. **A durable HITL gate at the transfer step**, decision set {approve, edit, reject}, authority policy requiring a finance role and dual approval above a threshold, timeout → auto-reject after 24h, full audit record on the event history. The gate is a durable interrupt the run sleeps on, woken by a validated signal.

## Key takeaways

- HITL is a **durable, resumable interrupt**, not a blocking call: the run persists "waiting at gate G," yields its worker, and resumes on a signal — so the wait survives an arbitrary delay and a process restart.
- The decision set is **approve / edit / reject / respond**, and each resumes the run differently. Specifying the *valid subset per gate* is the core design artifact.
- The gate carries a full **approval contract**: allowed decisions, authority policy, timeout-with-default, immutable audit record, a stable `(run_id, gate_id)` correlation key, and idempotent resume (first valid decision wins).
- Place gates by **blast radius** — gate the irreversible/high-consequence, never the cheap/reversible — and surface the proposal so the human can actually decide. The gate-placement map mirrors the idempotency side-effect list from Chapter 1.
