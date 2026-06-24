# Chapter 4 — The Discipline of Starting Simple

Anthropic's central guidance for agentic systems is a single sentence: *find the simplest solution possible, and only increase complexity when needed.* It sounds obvious. It is also the design rule teams most consistently violate, because every incentive — the demo, the résumé, the excitement of building an "agent" — pushes toward more autonomy than the problem requires. This chapter turns the slogan into a discipline: an explicit, auditable gate that complexity has to pass through.

## Complexity is a cost you must justify

The default question is backwards. Teams ask "what would make this more capable?" The architect asks "what is the *least* this can be and still meet the requirement?" Complexity is not free capability — it is a recurring liability:

- **Latency** rises with every added LLM call and loop iteration.
- **Cost** rises, and for model-driven control flow it rises *unpredictably*.
- **Debuggability** falls as control flow moves from code to model.
- **Failure surface** grows with every tool, sub-agent, and autonomous decision.
- **Evaluation burden** grows — you now have to score trajectories, not assert on outputs.

So the burden of proof sits on complexity, not simplicity. You do not justify *not* building an agent. You justify building one.

## The autonomy ladder

Think of architectural options as rungs. Always start on the lowest rung that could plausibly work, and climb only when you have evidence the current rung is insufficient.

```text
  ┌──────────────────────────────────────────────────────────┐
  │ 5. Autonomous agent (model owns control flow, loops)      │  most flexible,
  ├──────────────────────────────────────────────────────────┤  least predictable
  │ 4. Orchestrator-workers / evaluator-optimizer workflow    │      ▲
  ├──────────────────────────────────────────────────────────┤      │
  │ 3. Multi-step workflow (chaining, routing, parallel)      │   climb only
  ├──────────────────────────────────────────────────────────┤   on evidence
  │ 2. Single LLM call + retrieval / one tool                 │      │
  ├──────────────────────────────────────────────────────────┤      │
  │ 1. Single LLM call (well-prompted, maybe few-shot)        │  simplest,
  └──────────────────────────────────────────────────────────┘  most predictable
```

For many tasks, optimizing a single well-crafted call with retrieval and good examples outperforms a multi-agent system — at a fraction of the cost and with none of the debugging tax. Start at rung 1. Most production value lives on rungs 1–3.

## The upgrade gate

Climbing a rung requires passing a gate. Make it explicit; write it into your ADR. To move up, you must answer all four:

1. **What specifically fails at the current rung?** Name the failure with evidence — eval scores, error cases, real traces — not a hypothetical ("an agent *could* handle edge cases" is not evidence).
2. **Does the next rung demonstrably fix it?** Show that the added complexity addresses *that* failure, measured, not assumed.
3. **What does the upgrade cost?** Quantify the added latency, token spend (and its variance), and the new failure modes you are taking on.
4. **Is the improvement worth the cost for the volume?** A 3% quality gain that triples cost may be right for 100 high-stakes requests/day and wrong for 10 million low-stakes ones.

If you cannot answer (1) with a real failure, you are not allowed to climb. "It might need it later" is a YAGNI violation — build the simple version, instrument it, and let real failures pull you up the ladder.

## A test you can defend

For any proposed agentic component, force the decision through three questions:

| Test | Pass (stay simple) | Fail (justifies complexity) |
| --- | --- | --- |
| **Predictability test** — can you draw the full control-flow graph in advance? | Yes → workflow or fixed call | No → model must decide at runtime |
| **Evidence test** — do you have measured failures at the simpler rung? | No → stay put | Yes → consider climbing |
| **Economics test** — does the gain justify the cost at your volume? | No → stay put | Yes → climb is defensible |

A proposal must fail the predictability test *and* pass the evidence and economics tests before it earns a higher rung. Two out of three is not enough.

## Where complexity *is* justified

Starting simple is not an excuse to under-build. Genuine signals that you have outgrown a rung:

- Measured quality plateaus below requirement and the failures are concentrated in cases that demand runtime course-correction.
- The input space is provably open-ended — you keep discovering categories your routing table did not anticipate.
- The task value is high enough that the cost of autonomy is small relative to the cost of a wrong or incomplete answer.

When those hold, climbing is the right call — and because you instrumented the simpler version first, you can prove it.

## Documenting the decision

Every climb produces an artifact: an ADR that records the rung you chose, the rung you rejected, the failure evidence that justified the climb, and the cost you accepted. This is not bureaucracy — it is what lets the next architect (or the next you) understand *why* the system is as complex as it is, and challenge it when the inputs change. An agentic system whose complexity nobody can justify is a system nobody can safely simplify.

## Key takeaways

- The discipline: **start at the lowest rung that could work; climb only on measured evidence that the current rung fails.**
- Complexity is a liability — latency, unpredictable cost, lost debuggability, larger failure surface — and the burden of proof sits on it, not on simplicity.
- The **upgrade gate** is four questions: what fails, does the next rung fix it, what does it cost, is it worth it at your volume. No real failure, no climb.
- Record every climb as an ADR so the system's complexity stays justifiable — and reversible — as requirements change.
