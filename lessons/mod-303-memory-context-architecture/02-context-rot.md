# Chapter 2 — Diagnosing and Mitigating Context Rot

A long context window looks like a gift and behaves like a tax. The naive mental model — "bigger window, more the model knows, better the answer" — is wrong in a way that bites long-horizon agents specifically. As the window fills with turns, tool outputs, and retrieved chunks, the model's ability to *use* what's in there degrades. The answer doesn't just stop improving; it gets **worse**. This is **context rot**, and for an architect it is a failure mode to be measured, budgeted against, and mitigated by design — not a quirk to be discovered in production.

## What rot is, mechanically

Context rot is the measurable decline in agent quality as the context window grows longer and noisier, even when the relevant information is technically still present. It has three compounding sources:

- **Attention dilution.** Transformer attention is a finite resource spread across the window. A fact that occupied 100% of a 2K-token window competes with thousands of other tokens in a 150K-token window. The "lost in the middle" effect is the well-known instance: models attend most strongly to the start and end of a long context and least to the middle, so a critical fact buried at 60% depth is effectively dimmed.
- **Distraction by stale content.** Old tool outputs, abandoned reasoning paths, and resolved errors don't just take up space — they actively mislead. The model may re-engage a thread the user already closed, or cite a stale value it retrieved 30 turns ago instead of the fresh one. Noise is not neutral; it competes with signal.
- **Self-conditioning on its own history.** An agent reads its own prior turns. Early mistakes, verbose patterns, or a wrong assumption made at turn 5 get reinforced because the model conditions on a transcript that now "votes" for the error. Rot compounds: a slightly-off turn makes the next turn slightly more off.

The architectural consequence: **quality is a function of window occupancy, and the function is not monotonic.** There is a sweet spot — enough context to be grounded, not so much that signal drowns — and your job is to keep the agent in it for the whole run, not just the first few turns.

```text
  task
  quality
    ▲
    │        ╭───────────╮              the sweet spot is a
    │      ╭─╯           ╰──╮           band, not a ceiling
    │    ╭─╯                 ╰────╮
    │  ╭─╯                        ╰──────╮
    │ ╭╯  too little                     ╰─── rot: attention diluted,
    │╭╯   context (ungrounded)               stale content distracts,
    └┴──────────────────────────────────────────▶ tokens in window
         ↑ under-fill            ↑ over-fill
```

## How to diagnose it

You cannot mitigate what you cannot see. Rot is invisible in a single happy-path demo and obvious in a long-horizon eval. Three diagnostic moves an architect mandates:

### 1. Measure quality as a function of window length

Run the same agent on tasks of increasing length (more turns, more retrieved context) and plot a quality metric (task success, judge score from [mod-304](../mod-304-evaluation-harnesses/README.md)) against tokens-in-window or turn count. A flat or rising line means you're fine; a line that **peaks and falls** is rot, and the peak tells you where your budget should cap. This is a standard part of a long-horizon eval suite, not a one-off.

### 2. Probe with a needle-in-a-haystack and a "distractor" variant

The classic retrieval probe places a fact ("the magic number is 7,431") at a known depth and asks for it back. It tests positional recall. The more revealing architect's variant adds **distractors** — plausible-but-wrong facts elsewhere in the window — because real agent context is full of stale-but-related content. An agent that aces a clean needle test and fails the distractor variant is exactly the agent that will cite a stale tool output in production.

### 3. Instrument occupancy and attribute failures to it

Log, per turn: tokens in window, fraction that is stale (history older than N turns), fraction retrieved this turn, and turn index. When a run fails, you want to ask "was the window 90% full of 40-turn-old content?" and answer it from telemetry, not a guess. This is the same instrumentation you'll build in [exercise-01](exercises/exercise-01-context-budget-and-rot-analysis.md) and the tracing you'll formalize in [mod-305](../mod-305-observability-tracing/README.md).

## The mitigation levers

Every lever from [Chapter 1](01-context-engineering-strategies.md) is a rot mitigation when you read it through this lens. The framing changes from "save tokens" to "preserve signal."

| Lever | What it removes | Rot source it targets |
| --- | --- | --- |
| **Compaction** | Stale, verbose history | Distraction, dilution |
| **Structured notes** | Working state from the fragile transcript | Self-conditioning, dilution |
| **Sub-agent isolation** | A whole noisy exploration from the parent | Dilution, distraction |
| **JIT retrieval** | Pre-loaded content that's never used | Dilution |
| **Recency placement** | Burying the critical fact in the middle | Lost-in-the-middle |
| **Window reset / re-anchoring** | The accumulated drift, periodically | Self-conditioning |

Two levers in that table are specific to rot and worth drawing out:

- **Recency placement.** Because attention favors the edges, place the *most decision-critical* content where the model attends best: the durable instructions at the very front (also good for caching), and the *current* task framing and freshest facts near the end, right before the model acts. Don't bury the user's actual question under 40K tokens of retrieved chunks.
- **Re-anchoring / periodic reset.** For very long runs, the strongest mitigation is to *not* run forever in one window. Periodically compact hard, re-state the goal and the current plan from the structured notes, and effectively restart the working context from a clean, distilled anchor. This is "structured note-taking" used as a reset mechanism: the durable store is the source of truth, and the window is a disposable working copy you refresh before it rots.

## The architect's rot budget

Tie it back to the budget from Chapter 1. Rot mitigation is not free — every lever costs something (compaction costs a summarization call and is lossy; isolation costs latency; JIT costs round-trips). The architect sets a **rot budget**: an occupancy ceiling and a staleness ceiling, with named triggers when either is crossed.

```text
ROT BUDGET (per agent)
  Occupancy ceiling:  context never exceeds ___% of window at steady state
  Staleness ceiling:  no more than ___% of window is history older than ___ turns
  Critical-fact rule: the goal + freshest facts live in the last ___K tokens
  Trigger — occupancy crossed:  compact history to its budgeted slice
  Trigger — staleness crossed:  re-anchor from structured notes, drop old turns
  Trigger — long run (> N turns): hard reset working context from the store
```

The mindset shift is the deliverable: **more context is not more capability.** Past the sweet spot it is a liability. An architecture that treats the window as something to *keep clean* — actively evicting, re-anchoring, and capping occupancy — beats one that treats it as something to *fill*, every time the run gets long.

## Key takeaways

- **Context rot** is non-monotonic quality decline as the window grows: attention dilutes, stale content distracts, and the agent self-conditions on its own drift. Bigger windows do not mean better answers past a sweet spot.
- **Diagnose it deliberately**: plot quality vs. window length, run needle-in-a-haystack *with distractors*, and instrument per-turn occupancy and staleness so you can attribute failures to it.
- **Mitigate by preserving signal, not just saving tokens**: compaction, notes, isolation, and JIT retrieval all reduce rot; add **recency placement** (critical facts at the edges) and **periodic re-anchoring** from the durable store for very long runs.
- The deliverable is a **rot budget** — an occupancy ceiling, a staleness ceiling, and the triggers that fire when either is crossed.
