# Chapter 5 — Developer Experience: Adoption, Onboarding, and Feedback Loops

You can architect a flawless plugin, a watertight trust boundary, a lazy hydration strategy, and a clean event-driven integration — and still watch the platform die, because engineers tried it once, hit friction, and went back to doing it by hand. **A developer platform that nobody adopts is a failed platform, regardless of its technical merit.** Developer experience is not polish you add at the end; it is an architectural property you design for, and it is the explicit subject of the role this module targets. This chapter is about why platforms get bypassed and the DX architecture that prevents it.

## Why developer platforms get bypassed

Internal tooling fails adoption for predictable reasons, and each has an architectural answer:

| Failure mode | What it looks like | Architectural answer |
| --- | --- | --- |
| High activation cost | "I'd have to read a wiki and configure 6 things first" | one-command install; sane defaults |
| Untrustworthy output | agent did something dumb once; engineer never returned | guardrails + transparency, not just accuracy |
| Invisible value | "I don't know what it's good at" | discoverability; a short list of golden tasks |
| No feedback channel | annoyance has nowhere to go but Slack venting | in-loop feedback that reaches the platform team |
| Silent breakage | a model or API change quietly degrades it | the platform team watches eval/usage metrics |

The throughline: adoption is won or lost in the **first ten minutes** and **first failure**. Design for both.

## Onboarding: the first ten minutes

The activation path has to be near-frictionless, because every step before first value is a place engineers drop off.

- **One install, central source of truth.** The plugin (Chapter 1) is the lever: an engineer installs *one* plugin from the org marketplace and instantly has the subagents, skills, hooks, and internal-service MCP access the platform team curated. No per-person setup wiki, no copy-pasting config. The platform team versions and ships it; engineers consume it.
- **Sane, opinionated defaults.** The platform should do something useful out of the box with zero configuration. Defaults are an adoption decision: every required choice before first value is a drop-off point.
- **A golden-path first task.** Onboarding should hand the engineer *one concrete win* — "ask it to implement a small ticket" — not a feature tour. First value beats first completeness. People adopt tools that solved a real problem on day one.

## Documentation as part of the system

Docs for an agentic platform are not just human-facing; **some of your "documentation" is consumed by the agent itself.**

- **Skills are executable docs.** A `pr-conventions` skill *is* the documentation of how your org writes PRs — and the agent reads and applies it. Investing in clear skills pays twice: humans can read them, and the agent enforces them. This collapses the usual gap between "the wiki says X" and "the tooling does Y."
- **Human docs: task-oriented, not feature-oriented.** Engineers want "how do I get the agent to fix a flaky test?" not "reference for all 14 MCP tools." Lead with the golden tasks the platform is genuinely good at; be honest about what it is *not* good at yet (managing expectations is itself an adoption tool — overpromising and underdelivering is how you lose a skeptic permanently).
- **Make capabilities discoverable in-context.** The platform should be able to answer "what can you do here?" from inside the tool, so discovery does not require leaving the workflow to find a wiki.

## Continuous feedback loops

A platform that ships and then goes quiet decays — the model changes, an API changes, a convention drifts. The architecture needs **feedback loops in both directions**:

```text
   ENGINEERS                  PLATFORM TEAM
   ─────────                  ─────────────
  use the agent  ──telemetry──▶  usage + outcome metrics
       │                          (which tasks, success rate,
       │                           where it gives up)
       │◀──improvements──         │
       │   (new skills,           ▼
       │    fixed tools,    eval suite gates every
       │    better defaults) plugin change (mod-304)
       │                          │
  in-loop  ──thumbs / report──▶  triaged feedback →
  feedback     a bad run          turned into eval cases
```

- **Instrument adoption and outcomes.** Track which tasks engineers actually use the agent for, the success rate per task type, and *where it gives up or gets corrected*. This is the product signal that tells the platform team what to build next — and it reuses the observability instrumentation from the evaluation track.
- **Make bad runs reportable in-loop.** A one-keystroke "this run was bad" that captures the trajectory and routes it to the platform team beats hoping people file tickets. Every reported bad run becomes a regression **eval case** (the mod-304 discipline), so the same failure cannot silently return after the next model or API change.
- **Gate platform changes on evals.** The platform is software; ship it like software. A change to a skill, a tool, or the model version goes through the eval suite before rollout, so you catch a regression *before* engineers do — silent breakage is the fastest way to lose the trust you spent months earning.

## A worked DX plan

*"Roll out the SDLC platform to a 50-engineer org."* The DX plan:

| Phase | Move | Success signal |
| --- | --- | --- |
| Activate | one-command plugin install; 3 golden tasks documented | time-to-first-success < 15 min |
| Trust | guardrails + visible reasoning; honest "not good at yet" list | engineers return after first imperfect run |
| Discover | in-context "what can you do here?"; task-oriented docs | usage spreads beyond the pilot team |
| Feedback | one-key bad-run report → eval case; usage telemetry | bad runs become eval cases; no silent regressions |
| Sustain | eval-gated plugin releases; metrics reviewed weekly | adoption curve holds, doesn't decay |

Adoption is the deliverable, and it is measured — time-to-first-success, return-after-failure, usage spread, regression count. Producing this plan for a real rollout is [exercise-05](exercises/exercise-05-developer-tooling-adoption-and-dx.md).

## Key takeaways

- A developer platform nobody adopts is a **failed platform** regardless of technical merit; DX is an architectural property you design for, not end-stage polish.
- Adoption is won or lost in the **first ten minutes** (one-command install, sane defaults, a golden-path first win) and the **first failure** (guardrails, transparency, and honesty about limits beat raw accuracy).
- For an agentic platform, **some documentation is consumed by the agent** — skills are executable docs that pay twice; human docs should be task-oriented and honest about what the platform isn't good at yet.
- Build **bidirectional feedback loops**: instrument adoption and outcomes, make bad runs reportable in-loop and turn each into a regression **eval case**, and **gate every platform change on evals** so silent breakage never erodes hard-won trust.
- The deliverable is an **adoption plan with measured success signals** (time-to-first-success, return-after-failure, usage spread), not a launch announcement.
</content>
