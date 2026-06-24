# exercise-03: Tool vs. Sub-Agent Decomposition

**Estimated effort:** 3 hours

## Objective

Take a single non-trivial system — a **support-automation platform** — and decompose every capability into its correct home: **hardcoded path, tool, or sub-agent**. Then record the three or four most contestable boundary decisions as **architecture decision records (ADRs)**, so a reviewer can challenge your reasoning. The deliverable is a capability map plus ADRs, not an implementation.

## Background

This exercise covers material from:

- [Chapter 3 — Drawing the Boundaries: Tool, Sub-Agent, or Hardcoded Path](../03-tool-subagent-boundaries.md)
- [Chapter 4 — The Discipline of Starting Simple](../04-start-simple-discipline.md)

Anchor reference: Anthropic's [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (tools and the "keep it simple" guidance).

## Prerequisites

- You can articulate who decides and what context each boundary costs (Chapter 3's comparison table).
- Familiarity with the ADR format (context → decision → consequences). If new to ADRs, see the template under Starter guidance.

## The system

A support-automation platform that, for an inbound customer message, must: authenticate the requester; classify the issue; retrieve relevant account/order data; for hard cases, investigate across logs, docs, and prior tickets; draft a resolution; and send it (with a human approval gate for refunds above a threshold).

## Tasks

### 1. Enumerate the capabilities

- List every distinct capability the system needs (aim for 10–14). Be granular: "look up order" and "search knowledge base" are different capabilities.

### 2. Assign each capability a boundary

- For each, decide **hardcoded path / tool / sub-agent**, using Chapter 3's decision tree. Record *who decides* and *what context it costs* for each.
- Apply the default bias: hardcode unless model judgment is genuinely required; use a sub-agent only where a separate context window is justified (isolation, specialization, or parallel decomposition).

### 3. Draw the reference architecture

- Produce a single diagram showing the capabilities, their boundary types, and the data/control flow between them. Mark clearly where the model's judgment is fenced in and where deterministic code surrounds it.

### 4. Write the contestable decisions as ADRs

- Pick the **3–4 boundary calls most likely to be argued** (e.g., "is cross-system investigation a tool or a sub-agent?", "is classification a hardcoded router or a tool?", "is sending an email a hardcoded trigger or a tool the model may call?").
- Write an ADR for each: the alternatives considered, the decision, and the consequences (including the failure mode you are accepting).

### 5. Find the anti-patterns

- Identify at least one capability where the "obvious" choice is the wrong one per Chapter 3's anti-patterns (sub-agent-where-a-tool-would-do, tool-where-hardcoded-would-do, etc.), and show the correction.

## Starter guidance

Use this capability-map table and ADR template. The deliverable is the filled-in design, not code.

```text
CAPABILITY MAP

  Capability                 Boundary        Who decides       Context cost     Justification
  -------------------------  --------------  ----------------  ---------------  -----------------------
  Authenticate requester     hardcoded       engineer          none             security-critical, fixed
  Classify issue             tool/route      model: category   small result     judgment, compact return
  Look up order              tool            model: whether    small result     bounded capability
  Cross-system investigation sub-agent       parent: spawn     separate window  large context, distilled
  Send resolution email      hardcoded*      engineer (+human) none             act is code; gate the decision
  ...                        ...             ...               ...              ...
```

```text
ADR-00X: <the contestable decision>

  Status:        Accepted
  Context:       <what forces this decision; the competing pressures>
  Options:       A) <e.g., tool>   B) <e.g., sub-agent>   C) <e.g., hardcoded>
  Decision:      <chosen option>
  Rationale:     <tie to who-decides / context-cost / start-simple>
  Consequences:  <what you gain; the failure mode you accept; what would make you revisit>
```

## Acceptance criteria

You can demonstrate that:

- Every capability (10+) is assigned a boundary with *who decides* and *context cost* recorded.
- The reference-architecture diagram shows boundaries and the fenced-in model judgment.
- 3–4 ADRs document the contestable calls with alternatives and accepted consequences.
- At least one anti-pattern is identified and corrected, with the reasoning shown.
- The decomposition reflects the start-simple bias — sub-agents appear only where isolation/specialization/parallelism genuinely justify them.

## Reflection

In `NOTES.md`:

1. Which capability was hardest to place, and what tipped the decision in the end?
2. Where did you resist the temptation to use a sub-agent, and what would have to change for that capability to deserve one?
3. For one ADR, describe the future input change that would make you reverse the decision.

## Stretch goals

- Add a **cost annotation** to the architecture: estimate relative token/latency cost per boundary and identify the single most expensive capability. Is its boundary justified?
- Introduce a new requirement — "handle messages in five languages" — and show which boundaries change (does classification become routing? does a translation tool appear?).
- Map each tool you defined to a minimal **tool spec** (name, typed inputs, output shape, error behavior) and note which tool, if poorly specified, would most degrade the system (ties tool-design quality to architecture).
