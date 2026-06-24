# exercise-04: Platform Adoption and DX Plan

**Estimated effort:** 3 hours

## Objective

Design the **developer-experience and adoption architecture** for an internal agent platform: the activation path, the trust-under-failure design, the documentation-that-pays-twice strategy, the versioning policy, and the eval-gated feedback loop. The deliverable is an adoption plan an architect could hand to a platform team — concrete enough that "users adopt it instead of routing around it" is an engineered outcome, not a hope.

This is an **architecture** exercise. You will not build the platform; you will design the DX that decides whether it lives.

## Background

This exercise covers material from:

- [Chapter 4 — Platform Developer Experience and Adoption](../04-platform-developer-experience-and-adoption.md)
- [Chapter 1 — The Extension Model Across Platforms](../01-extension-model-across-platforms.md) — the versioned bundle is the activation and versioning lever.

## The scenario

Your org shipped an internal agent platform six weeks ago. Telemetry says: many people tried it once, few came back. Exit interviews reveal two patterns:

> 1. New users spent 30+ minutes wiring tools and hunting credentials before getting any result, and most quit before the first one.
> 2. Of those who got a result, several hit a **confusing wrong answer** early, lost trust, and went back to doing the work by hand. One bad action was **irreversible** and caused a minor incident.

Diagnose against the two decisive moments, then design the fix. The "users" may be developers, on-call engineers, support staff, or analysts — your plan should not assume they write code.

## Tasks

### 1. Diagnose against the two decisive moments

- Map each exit-interview pattern to **activation** (the first ten minutes) or **trust under failure** (the first failure), per [Chapter 4](../04-platform-developer-experience-and-adoption.md).
- Name the root cause of each — not the symptom. ("30 minutes of setup" → an *activation/setup-friction* failure; "irreversible bad action" → a *reversibility-ladder* failure that compounded the trust loss.)

### 2. Design the activation path

- Specify the **first-ten-minutes** experience: one-step install via the versioned bundle ([Chapter 1](../01-extension-model-across-platforms.md)), a curated first task that obviously pays, and the sane defaults that make it work out of the box.
- State the metric you would track to know activation improved (a new user reaching a valuable result fast) — and the vanity metric you would *refuse* to optimize.

### 3. Design trust under failure

- Specify the three trust-preserving properties from [Chapter 4](../04-platform-developer-experience-and-adoption.md): **legibility** (the user can see what the agent did and why), **reversibility by default** (failures land on the safe side of the ladder), and a **blameless, frictionless report path** that visibly leads somewhere.
- Address the incident specifically: how does your design keep an early bad action *reversible* so a wrong answer costs trust once instead of causing an incident?

### 4. Design documentation that pays twice

- Identify two conventions/procedures in this platform that should be authored as **skills**, and show how each serves *both* a human (onboarding/convention) and the agent (loaded behavior), per [Chapter 4](../04-platform-developer-experience-and-adoption.md).
- State where these skills live (the versioned bundle) and why that keeps doc and behavior from drifting apart.

### 5. Design the versioning policy and feedback loop

- **Versioning:** state the rules that let the platform evolve without surprise — versioned bundle, no silent behavior changes (especially on the privileged/irreversible tier), reversible rollout.
- **Feedback loop:** design the path from a reported bad run to a **permanent regression eval case** to an **eval gate** on every future platform change, per [Chapter 4](../04-platform-developer-experience-and-adoption.md). Show why this makes the platform improve under model/API change instead of regressing.

```text
   bad run reported ─▶ captured ─▶ regression eval case ─▶ eval suite
                                                              │
                          every platform change GATED ◀───────┘
```

## Starter guidance

A diagnosis frame to react to:

```text
   pattern 1 (30-min setup, quit early)   → ACTIVATION failure
   pattern 2 (wrong answer → routed around)→ TRUST-UNDER-FAILURE failure
   pattern 2b (irreversible bad action)    → REVERSIBILITY-LADDER failure
```

A feedback-loop shape to adapt:

```text
   report (1 frictionless action)
     → capture {inputs, context, tools called, bad output}
     → distill to a regression eval case
     → add to eval suite
     → gate every model/API/skill/tool change on it
```

You do **not** need the full tool/trust design (exercise-02) or integration interfaces (exercise-03) here — assume them and focus on adoption, DX, versioning, and feedback.

## Acceptance criteria

You can demonstrate that:

- Each exit-interview pattern is mapped to activation or trust-under-failure with a *root cause*, not a symptom.
- The activation path is one-step-install + curated-first-task + sane-defaults, with a real activation metric named and a vanity metric explicitly refused.
- Trust-under-failure design covers legibility, reversibility-by-default, and a frictionless report path, and specifically prevents the irreversible-action incident from recurring.
- At least two conventions are authored as **skills** that pay twice, kept in the versioned bundle.
- The versioning policy forbids silent behavior changes on the privileged tier, and the **feedback loop turns every reported bad run into an eval-gated regression case**.

## Reflection

In `NOTES.md`:

1. Which fix would move adoption most in the first month — activation or trust — and why?
2. Walk one specific bad run from report to permanent eval case, and show what would have happened on the *next* model update without that gate.
3. Which "feature" on the roadmap matters less than these two moments, and what would you cut to fund the DX work instead?

## Stretch goals

- Design the **activation and trust dashboards**: the few real signals (time-to-first-value, return-after-failure rate) you would instrument, and the vanity metrics you would keep off them.
- Generalize the whole DX plan to a **non-developer** user population (on-call, support, analysts) and note what changes about "the first ten minutes" when the first use happens under incident pressure ([Chapter 5](../05-beyond-sdlc-generalizing-the-platform.md)).
- Design how a **widened tool scope** flows through *both* the security-review path and the versioning path as a single reviewed event.
