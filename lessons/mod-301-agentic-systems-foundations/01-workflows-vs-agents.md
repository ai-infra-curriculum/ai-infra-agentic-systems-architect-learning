# Chapter 1 — Workflows vs. Agents: The Foundational Choice

Anthropic's *Building effective agents* draws a line that most teams blur: the difference between a **workflow** and an **agent**. Both are "agentic systems," but they are architecturally distinct, and choosing the wrong one is the most expensive mistake an architect makes at this layer.

- A **workflow** is a system where LLM calls and tools are orchestrated through **predefined code paths**. The control flow is fixed by the engineer; the model fills in steps. You can draw the whole graph before the system runs.
- An **agent** is a system where the **model itself directs the control flow** — it decides which tools to call, in what order, and when it is done. You cannot draw the full graph in advance, because the model draws it at runtime.

The distinction is not "uses an LLM" vs. "doesn't." It is *who owns the control flow* — the code, or the model.

## The tradeoff in one axis

```text
  PREDICTABLE                                          FLEXIBLE
  cheap / fast / debuggable          adaptive / open-ended / costly
  │                                                            │
  ▼                                                            ▼
 [ single   ]   [ workflow:        ]   [ workflow:    ]   [ autonomous ]
 [ LLM call ]   [ chain / route /  ]   [ orch-workers ]   [ agent      ]
 [          ]   [ parallelize      ]   [ eval-optimize]   [ (loop)     ]
  └──────────────── fixed control flow ──────────┘   └─ model-driven ─┘
```

Every step rightward buys **flexibility** — the system can handle inputs you did not anticipate — and pays in **predictability**: more latency, more tokens, more failure modes, and a control flow you can no longer fully enumerate for testing, auditing, or debugging. The architect's job is to move *only as far right as the problem demands*, and not one step further.

## When a workflow is the right answer

Choose a workflow when the task decomposes into **knowable steps**:

- You can enumerate the steps in advance, even if each step needs an LLM (e.g., "translate, then summarize, then format").
- The valid paths through the system are few and known. Branching is fine — routing to one of five handlers is still a workflow.
- You need **predictable cost and latency** per request, or strict auditability (regulated domains often mandate this).
- Failures must be **reproducible** for debugging. Fixed paths replay deterministically; agent trajectories do not.

## When an agent earns its cost

Choose an agent only when **the path cannot be known in advance**:

- The number and order of steps genuinely depends on the input and on intermediate results (open-ended research, multi-step debugging, "do whatever it takes to resolve this ticket").
- The decision space is too large to encode as branches — you cannot write the routing table because you do not know the routes.
- The task benefits from the model **course-correcting** based on tool results (it tried X, X failed, it should now try Y — and you cannot pre-script that).

If you can describe the solution as a flowchart, build the flowchart. Reach for an agent when the flowchart would need to be drawn at runtime.

## Decision criteria

| Question | Lean workflow | Lean agent |
| --- | --- | --- |
| Can you enumerate the steps in advance? | Yes | No |
| Is the path the same shape every run? | Yes | No |
| Do you need predictable cost/latency? | Yes | Can tolerate variance |
| Must failures replay deterministically? | Yes | No |
| Does the model need to course-correct mid-task? | No | Yes |
| Is the input space bounded and known? | Yes | Open-ended |

If the answers split, the bias is toward the workflow. A workflow that occasionally escalates to an agent for the hard 5% is almost always cheaper and safer than an agent that handles 100% of traffic autonomously.

## The cost of autonomy is not just tokens

Architects under-price autonomy because they think the cost is tokens and latency. Those are the *visible* costs. The hidden costs are larger:

- **Debuggability collapses.** A workflow failure points at a line of code. An agent failure points at a trajectory you have to reconstruct, often non-reproducibly.
- **The blast radius grows.** A model that owns control flow can take actions you did not anticipate. Every tool you give it is a new way for a bad trajectory to do damage.
- **Evaluation gets harder.** You can unit-test a workflow step. Evaluating an agent means scoring whole trajectories against fuzzy rubrics (covered in mod-304).
- **Cost variance is unbounded** unless you cap it. An agent can loop, retry, and fan out far beyond your estimate.

These costs are why "start simple" (Chapter 4) is a discipline and not a slogan.

## Worked judgment

*"Generate a weekly compliance report from three data sources."* — The sources are known, the format is fixed, the steps are the same every week. This is a **prompt-chaining workflow**, not an agent. Making it an agent adds cost and a non-reproducible audit trail for zero benefit.

*"Investigate why this customer's account is in a bad state and propose a fix."* — You cannot enumerate the investigation path; it depends on what each lookup reveals. This is a genuine **agent** task — the model must course-correct based on tool results.

## Key takeaways

- The workflow/agent line is about **who owns control flow** — fixed code paths (workflow) vs. the model deciding at runtime (agent).
- Move right on the predictability→flexibility axis **only as far as the problem forces you**; bias toward workflows.
- If you can draw the flowchart in advance, build the flowchart; reach for an agent only when the path must be drawn at runtime.
- The cost of autonomy is debuggability, blast radius, evaluation difficulty, and cost variance — not just tokens.
