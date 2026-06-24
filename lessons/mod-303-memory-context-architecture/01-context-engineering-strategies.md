# Chapter 1 — Context Engineering as a Token Budget

Prompt engineering asks "what words do I put in the prompt?" **Context engineering** asks a harder, system-level question: *of everything that could be in the model's window on this turn, what is the smallest high-signal set that produces good behavior — and how does the system keep it that way over a long run?* For an architect, the reframing is the whole game. The context window is a finite, shared, contended resource. You do not tune it once; you design a regime that governs how it is spent across every turn, every sub-agent, and every session.

This chapter treats the window as a **budget** and the four core strategies — compaction, structured note-taking, sub-agent isolation, and just-in-time retrieval — as the line items you allocate against it.

## The window is a budget, not a bucket

A model has a maximum context (say 200K tokens). That is *capacity*, not *budget*. Your usable budget is smaller and you set it deliberately, because two forces push the other way:

- **Cost and latency scale with tokens in the window.** Every token in context is paid for on every turn it survives. A 150K-token window that lingers for 40 turns is not 150K tokens of spend — it is closer to 150K × 40, minus whatever caching reclaims.
- **Quality degrades before you hit the limit.** Effective attention thins out long before the hard cap. Loading "everything just in case" actively makes the agent worse — the subject of [Chapter 2](02-context-rot.md).

So the architect's first artifact is a **budget**: a per-turn allocation, in tokens, across the components that compete for the window.

```text
  Context window capacity: 200,000 tokens
  ┌───────────────────────────────────────────────────────────────┐
  │ system + tools    │ working set │ retrieved │ history  │ head- │
  │ (stable, cached)  │ (this task) │ (JIT)     │ (compact)│ room  │
  │ ~8K               │ ~20K        │ ~25K      │ ~40K     │ rest  │
  └───────────────────────────────────────────────────────────────┘
   ▲ rarely changes    ▲ changes      ▲ pulled    ▲ summarized  ▲ never
     → cache it           per task       on demand   on a trigger   fill
```

Two budget disciplines matter more than the exact numbers:

1. **Reserve headroom.** Never design to fill the window. Leave room for the model's own reasoning and for a turn that retrieves more than expected. A budget that targets ~60–70% of capacity at steady state survives spikes; one that targets 95% wedges the first time a tool returns a large payload.
2. **Separate the stable from the volatile.** Put the parts that rarely change (system prompt, tool definitions, durable instructions) at the front so prompt caching can amortize them, and keep the churning parts (history, retrieved chunks) behind them. This is where the budget meets [mod-307](../mod-307-cost-latency-architecture/README.md): cache strategy *is* budget strategy.

## The four line items

### Compaction — summarize history before it crowds you out

As a session runs, the transcript grows: tool calls, their outputs, the model's reasoning. Most of it is stale within a few turns. **Compaction** replaces a span of raw history with a model-written (or rule-written) summary that preserves decisions, open threads, and task-critical facts while discarding the verbatim scaffolding.

The architectural decisions are not "how do I summarize" but:

- **The trigger.** Compact on a token threshold (e.g., when history exceeds its budgeted slice), on a turn count, or at natural task boundaries. Threshold-based is the safe default; boundary-based is cleaner when the task has phases.
- **The fidelity contract.** What must *never* be lost in a compaction? Name it explicitly — the user's actual goal, IDs and file paths, irreversible actions already taken, unresolved errors. A compaction that drops the goal is worse than no compaction.
- **Reversibility.** Can the agent recover detail it compacted away? If the raw transcript is persisted (it should be — see [Chapter 3](03-memory-tiers-and-state.md)), compaction is lossy *in context* but recoverable *from store*. That distinction lets you compact aggressively.

You will design and defend a compaction trigger in [exercise-03](exercises/exercise-03-compaction-and-notetaking-strategy.md).

### Structured note-taking — a scratchpad outside the window

Compaction fights the window from the inside. **Structured note-taking** sidesteps it: the agent writes durable state to a structured artifact *outside* the context — a `NOTES.md`, a task ledger, a to-do list — and reads back only the slice it needs. The window holds a pointer and a summary; the store holds the detail.

This is the single highest-leverage move for long-horizon agents. An agent that maintains an explicit plan file and checks items off survives compaction, survives a context reset, and survives a sub-agent handoff, because its working state does not live in the fragile transcript. The architectural question is the **schema**: a free-text note rots as fast as the transcript; a typed ledger (`{step, status, artifact, blocker}`) stays useful because it is queryable and prunable.

### Sub-agent isolation — spend a fresh budget, return a summary

A sub-agent runs in its **own** context window. The parent spawns it with a narrow instruction, the sub-agent burns its own budget on the messy work (reading twenty documents, exploring a codebase), and returns a **distilled** result — a summary, not its transcript. The parent's window pays for the summary, not the exploration.

This is why isolation belongs in a *context* chapter, not just an orchestration one ([mod-302](../mod-302-multi-agent-orchestration/README.md) covers the coordination mechanics). As a budget instrument, a sub-agent converts an unbounded, high-token exploration into a bounded, low-token result the parent can afford. The architectural rule: **a sub-agent's return value is part of the parent's budget; design the return contract (length cap, required fields) the way you'd design an API response, not a chat reply.**

### Just-in-time (JIT) retrieval — load on demand, not up front

The opposite of "stuff everything in the prompt" is to keep lightweight **references** in context — file paths, record IDs, query handles — and let the agent fetch the underlying content only when a step actually needs it. This mirrors how an engineer works: you don't memorize the codebase, you `grep` and open the file when you reach it.

JIT retrieval trades a little latency (an extra tool call) for a lot of budget (you never pay for content you don't use) and a lot of freshness (you fetch the current value, not a snapshot baked in at session start). The cost is **autonomy risk**: an agent that must retrieve everything itself can retrieve the wrong thing, or loop. The architect's job is to decide the split — what is pre-loaded because it is always needed (the budget calls this the "working set"), versus what is referenced and fetched on demand. A hybrid is almost always right: pre-load the small, stable, always-needed core; reference everything else.

## How the line items interact

These four are not independent knobs — they trade against each other, and the architecture is the *combination*:

```text
                       pressure on the window
                               │
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                      ▼
   too much history      too much detail        too much breadth
        │                      │                      │
   ┌────┴────┐            ┌─────┴─────┐          ┌─────┴─────┐
   │COMPACT  │            │NOTE-TAKE  │          │ISOLATE in │
   │history  │            │to a store │          │a sub-agent│
   └────┬────┘            └─────┬─────┘          └─────┬─────┘
        └──────────────┬───────┴───────────────┬──────┘
                       ▼                        ▼
                  JIT-RETRIEVE the slice you actually need next
```

A worked example — a coding agent on a multi-hour task:

- **JIT retrieval** keeps source files out of the window until a step opens them.
- **Structured note-taking** holds the plan and progress in a `TASK.md` the agent updates, so its working state survives anything.
- **Sub-agent isolation** sends "investigate why test X fails" to a child that reads the failing test, the implementation, and the logs in *its* window and returns a two-paragraph diagnosis.
- **Compaction** fires when the conversation history crosses its budgeted slice, summarizing the early exploration into "explored modules A/B; root cause in C; see TASK.md."

No single strategy is sufficient; the budget is what tells you which one to reach for when the window gets tight.

## Starter guidance — the context budget worksheet

Use this as the artifact you produce for any agent. Fill the right column in tokens; the percentages are a sane starting allocation for a 200K window targeting ~65% steady-state fill.

```text
CONTEXT BUDGET WORKSHEET
Agent: ____________________   Model window: __________ tokens
Steady-state target: ~65% of window = __________ tokens (your budget)

Component             Strategy            Budget    Cache?  Eviction rule
--------------------- ------------------- --------- ------- -------------------------
System + instructions stable, front       ~4%       yes     never
Tool definitions      stable, front       ~3%       yes     never
Working set (task)    pre-loaded          ~10%      no      replace per task/phase
Structured notes      note-taking (ref)   ~3%       no      summary only; detail in store
Retrieved content     JIT retrieval       ~13%      no      drop after use
Conversation history  compaction          ~20%      partial summarize at threshold
Sub-agent returns     isolation           ~5%       no      distilled summary only
Headroom (reasoning)  reserved            ~35%      n/a     never fill
--------------------- ------------------- --------- ------- -------------------------

Triggers to specify:
  - Compaction fires when: history slice > ____ tokens  OR  task phase ends
  - Sub-agent spawned when: a sub-task needs > ____ tokens of its own exploration
  - JIT vs. pre-load: pre-load if needed on > ____% of turns, else reference
```

A budget you can defend line-by-line — *why* each component gets its slice, *when* it gets evicted, *whether* it caches — is the deliverable. The numbers will move per workload; the discipline of allocating, capping, and naming an eviction rule for every line item is what makes context an engineered system instead of an emergent mess.

## Key takeaways

- Context engineering is **budget allocation**, not prompt wording: decide, per turn, the smallest high-signal set the model needs, and govern it across the whole run.
- The window is a contended resource — reserve headroom (~30–40%), and order stable-before-volatile so caching can amortize the front.
- The four line items — **compaction** (shrink history), **note-taking** (offload to a store), **sub-agent isolation** (spend a fresh budget, return a summary), **JIT retrieval** (load on demand) — trade against each other; the architecture is the combination, chosen by where the window is under pressure.
- The deliverable is a **budget worksheet**: a token allocation with a cache flag and an eviction rule for every component.
