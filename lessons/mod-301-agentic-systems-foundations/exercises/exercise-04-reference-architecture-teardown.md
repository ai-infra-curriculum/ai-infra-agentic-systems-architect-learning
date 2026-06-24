# exercise-04: Reference Architecture Teardown

**Estimated effort:** 3 hours

## Objective

Reverse-engineer a **real, published agentic system** down to its design decisions, then critique it as an architect: name the patterns it uses, locate its tool/sub-agent/hardcoded boundaries, and judge whether each complexity choice was justified by the "start simple" gate. The deliverable is a teardown document — diagram, pattern identification, boundary map, and a reasoned critique with at least one defensible alternative.

## Background

This exercise covers material from all four chapters:

- [Chapter 1 — Workflows vs. Agents](../01-workflows-vs-agents.md)
- [Chapter 2 — The Orchestration Pattern Catalog](../02-orchestration-patterns.md)
- [Chapter 3 — Drawing the Boundaries](../03-tool-subagent-boundaries.md)
- [Chapter 4 — The Discipline of Starting Simple](../04-start-simple-discipline.md)

Use Anthropic's [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) as the primary teardown subject — it documents an orchestrator-worker architecture, its token economics, and its failure modes in unusual detail. You may instead choose another *publicly documented* agentic system if you can cite enough detail to support the teardown.

## Prerequisites

- You have completed exercises 01–03 (you will reuse the decision matrix, the pattern catalog, and the boundary vocabulary).
- Ability to read an engineering write-up and extract architecture from prose.

## Tasks

### 1. Reconstruct the architecture

- From the source material, draw the system's architecture: the agents/components, the control flow, and where work fans out and rejoins.
- Mark the boundary where control flow is fixed-path vs. model-driven.

### 2. Identify the patterns

- Name which of the five orchestration patterns the system uses, and where. Most non-trivial systems compose more than one.
- Identify what the system chose **not** to do (e.g., why orchestrator-workers and not a single agent? why a separate sub-agent for a sub-task?).

### 3. Map the boundaries

- For the major capabilities, label hardcoded / tool / sub-agent. Note the context-economics the source describes (e.g., why sub-agents return distilled results to protect the lead's context, and the token-multiplier the authors report).

### 4. Critique against "start simple"

- For each significant complexity choice, ask the upgrade-gate questions from Chapter 4: what failed at the simpler rung, did the complexity demonstrably fix it, what did it cost, was it worth it? Use the authors' own stated evidence where available.
- Flag any choice you find **under-justified** by the published evidence, and say what evidence would settle it.

### 5. Propose an alternative

- Design one defensible alternative architecture (e.g., "this could have been a routing workflow with a single agent escalation for the hard tail") and state the conditions under which your alternative would be the better call.

## Starter guidance

Use this teardown structure. It is an analysis document; the only "code" is the architecture sketch.

```text
TEARDOWN: <system name + source link>

  1. Architecture sketch
     <ASCII diagram: components, control flow, fan-out/rejoin,
      and the fixed-path | model-driven boundary marked>

  2. Patterns in use
     - <pattern> at <where> — evidence: <quote/figure from source>
     - Composition: <how patterns nest>
     - Chose NOT to: <simpler option rejected> — stated reason: <...>

  3. Boundary map
     Capability            Boundary      Context economics noted by authors
     -------------------   -----------   ----------------------------------
     <...>                 <...>         <...>

  4. Start-simple critique (per major complexity choice)
     Choice: <...>
       What failed simpler:   <author evidence or "not stated">
       Demonstrably fixed?:   <yes/no + evidence>
       Cost accepted:         <latency/tokens/failure surface>
       Verdict:               <justified | under-justified>

  5. Alternative architecture
     <sketch + the conditions under which it wins>
```

## Acceptance criteria

You can demonstrate that:

- The architecture is reconstructed as a diagram with the fixed-path/model-driven boundary marked.
- Every orchestration pattern present is named and located, with at least one cited piece of evidence from the source.
- The boundary map covers the major capabilities and references the source's stated context-economics.
- Each major complexity choice is run through the Chapter 4 upgrade gate, with at least one choice explicitly judged justified or under-justified on evidence.
- A defensible alternative architecture is proposed with the conditions under which it would win.

## Reflection

In `NOTES.md`:

1. What is the single design decision in this system you would most want to challenge in a design review, and what evidence would you ask for?
2. Where did the authors' reported costs (token multiplier, latency) change your read of whether the complexity was worth it?
3. If you were forced to ship a simpler version of this system for 100x the request volume, what would you cut first, and what would break?

## Stretch goals

- Run a **second** teardown on a contrasting system (a deliberately simple workflow-based product) and compare what each team optimized for.
- Convert your critique into a one-page **design-review checklist** an architect could apply to any agentic system write-up.
- Re-draw the architecture as the **lowest rung on the autonomy ladder** (Chapter 4) that you believe could still meet the system's stated goals, and list the evidence you would need to prove it sufficient.
