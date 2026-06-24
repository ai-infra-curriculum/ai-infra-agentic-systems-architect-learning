# exercise-05: Developer Tooling Adoption and DX

**Estimated effort:** 2 hours

## Objective

Design the developer-experience architecture that gets the platform *adopted* — and keeps it adopted. You will produce an adoption plan with measured success signals: a frictionless onboarding path, a documentation strategy (including the docs the agent itself consumes), and bidirectional feedback loops that turn bad runs into eval cases and gate every platform change on evals. The deliverable is a rollout plan a platform team could execute and measure.

## Background

This exercise covers material from:

- [Chapter 5 — Developer Experience: Adoption, Onboarding, and Feedback Loops](../05-developer-experience-and-adoption.md)

The premise: **a developer platform nobody adopts is a failed platform, regardless of technical merit.** Adoption is won or lost in the first ten minutes and the first failure.

## Prerequisites

- Read Chapter 5.
- The platform from exercises 1–4 (or any realistic internal agentic SDLC platform) as the thing you're rolling out.
- Familiarity with the eval-gating idea from the evaluation track (mod-304) — bad runs become regression cases.

## The scenario

Roll out the SDLC platform to a 50-engineer org that is *skeptical* — a previous internal tool flopped, and people default to doing things by hand.

## Tasks

### 1. Diagnose the bypass risks

- For each adoption failure mode (high activation cost, untrustworthy output, invisible value, no feedback channel, silent breakage), state how it would manifest in *this* skeptical org and the architectural answer you'll deploy.

### 2. Design the first ten minutes

- Specify the activation path: one-command plugin install, the sane defaults that work with zero config, and the single golden-path first task that delivers value on day one. Set a target time-to-first-success.

### 3. Design the documentation strategy

- Distinguish agent-consumed docs (skills as executable conventions) from human docs (task-oriented, honest about limits). List the 3–5 golden tasks the platform is genuinely good at, and what you'll explicitly say it's *not* good at yet.

### 4. Design the feedback loops

- Specify both directions: telemetry (which tasks, success rate, where it gives up) flowing to the platform team, and a one-keystroke in-loop bad-run report that captures the trajectory and becomes a regression eval case.

### 5. Design eval-gated releases

- Specify that changes to skills, tools, or the model version pass the eval suite before rollout, so regressions are caught before engineers hit them. Connect this to the bad-run-to-eval-case pipeline.

### 6. Define success signals

- Produce the rollout table: `phase | move | success signal`, with *measurable* signals (time-to-first-success, return-after-failure rate, usage spread, regression count).

## Starter guidance

Target rollout table:

```text
| Phase    | Move                                            | Success signal                  |
| -------- | ----------------------------------------------- | ------------------------------- |
| Activate | one-command install; 3 golden tasks documented  | time-to-first-success < 15 min  |
| Trust    | guardrails + visible reasoning; honest limits   | engineers return after first    |
|          |                                                 |   imperfect run                 |
| Discover | in-context "what can you do here?"; task docs   | usage spreads beyond pilot team |
| Feedback | one-key bad-run report → eval case; telemetry   | bad runs become eval cases      |
| Sustain  | eval-gated releases; weekly metric review       | adoption holds, doesn't decay   |
```

A trust-recovery note: the *first failure* matters more than the average. Design what the agent does when it's unsure (ask, show reasoning, decline) so a skeptic's first imperfect run doesn't end the relationship.

## Acceptance criteria

You can demonstrate that:

- Each bypass risk has a concrete manifestation in the skeptical org and a matching architectural answer.
- The activation path is one-command with sane defaults and a single golden first task, with a target time-to-first-success.
- The doc strategy separates agent-consumed skills from human task-oriented docs and is honest about current limits.
- The feedback design is bidirectional and routes every bad run into a regression eval case.
- Releases are eval-gated, and every rollout phase has a *measurable* success signal.

## Reflection

In `NOTES.md`:

1. For a skeptical org, is the first-ten-minutes problem or the first-failure problem the bigger adoption risk? Defend your answer.
2. Why do skills count as documentation that "pays twice," and what's the cost if your skills are sloppy?
3. What metric, if it started trending the wrong way, would be your earliest warning that adoption is decaying — before usage numbers drop?

## Stretch goals

- Design a champions/pilot-team rollout: who gets it first, and how their feedback shapes the broader launch.
- Add a "platform changelog" aimed at engineers (not just the team), and decide what belongs in it to build trust vs. what's noise.
- Specify the weekly metric review: which 3–4 numbers the platform team looks at, and the action each one triggers when it moves.
</content>
