# exercise-05: Data-Handling and Residency Design

**Estimated effort:** 3 hours

## Objective

Design the **privacy, residency, and retention controls** for an agentic platform's data handling — **parameterized by regime**, not hardcoded to any single statute. The deliverable is a data-classification scheme, a data-flow boundary map with minimization points, a residency map, a per-class retention schedule, and a **regime-profile** object that re-tunes all of the above for a new regime without changing the architecture.

This is an **architecture** exercise. You design the data-handling controls and their parameterization, not a working data pipeline.

## Background

This exercise covers material from:

- [Chapter 2 — Regulated Domain as Architecture](../02-regulated-domain-as-architecture.md) — privacy, residency, retention primitives.
- [Chapter 3 — Multi-Regime Mapping](../03-multi-regime-mapping.md) — parameterizing one architecture across regimes.

## The scenario

> An agentic platform ingests records about individuals, retrieves context via RAG, passes context into model prompts and tool calls, and writes logs/traces. It must be deployable under **different regimes** over time (the four peer regimes of Chapter 3). The design must not bake in one regime's thresholds — a new regime should be a new *profile*, not a new build.

The whole point: build the data-handling **mechanism** once; set the **parameters** per regime.

## Tasks

### 1. Design the data-classification scheme

- Define the sensitivity classes the platform tags at ingest (e.g., identifiable, sensitive-category, internal, public) and why classification is the precondition for *every* downstream control, per [Chapter 2](../02-regulated-domain-as-architecture.md).
- State that the *class labels* and the *"what counts as identifiable"* rule are **regime parameters**, while the tagging mechanism is fixed.

### 2. Map the data-flow boundaries and minimization points

- Draw the data flow (ingest → agent core → model prompt → tools/sub-agents → egress → logs/traces) and place a **minimization control** (strip/mask/tokenize/redact) at each boundary where data reaches a component that should not see it in the clear.
- Explicitly include the **model prompt**, **tool calls**, and **logs** as boundaries — these are the most-forgotten ones.

```text
   ingest ─(classify)─▶ agent core ─(minimize)─▶ model prompt
                            │
                            ├─(scope+minimize)─▶ tools / sub-agents ─▶ egress
                            └─(minimize)───────▶ logs / traces
```

### 3. Design the residency map

- For each data class, specify which region(s) it may live and be processed in — and treat the **model inference endpoint**, the **vector/document store**, **tool backends**, and the **trace pipeline** all as in-scope (the common miss).
- Specify the **crossing controls**: where data *must* cross a boundary, the crossing is explicit, logged, and minimized. State when a regime forces a **per-region deployment** of the whole stack.
- Make the **allowed-region set** a regime parameter.

### 4. Design the per-class retention schedule

- For each data class (input, retrieved context, prompts/completions, logs, run state, audit records), set a **minimum** and **maximum** lifetime and a **destruction method** — noting that audit records may need to outlive the raw data they describe.
- Specify automated lifecycle enforcement (TTLs/deletion jobs that actually fire) and a **right-to-erasure** flow that can find and remove a subject's data across memory, store, logs, and run state — enabled by the classification from task 1.
- Make the **schedules** regime parameters.

### 5. Define the regime-profile object

- Specify the concrete parameter object that re-tunes the whole design for a regime: identifiable-rule, class label set, allowed-region set, retention schedules, gate authority (reference exercise-02). Show the *same architecture* under two different profiles (pick two of the four peer regimes) to prove nothing structural changes — only parameters.

## Starter guidance

The parameterization principle to instantiate:

```text
   FIXED MECHANISM (build once)
     classify · minimize-at-boundary · region-pin · per-class lifecycle
        ▲ configured by ▼
   REGIME PROFILE (parameters)
     identifiable_rule · class_labels · allowed_regions ·
     retention_schedules · gate_authority
```

A retention-row shape to adopt:

```text
   data class: prompts/completions
   min: (audit need)   max: (privacy limit)   destroy: secure delete
   note: shorter than audit records; do not conflate the two stores
```

You do **not** design the audit-trail spec itself here (exercise-04) — but note that the retention schedule you set for audit records is consumed by it.

## Acceptance criteria

You can demonstrate that:

- The classification scheme is defined, and the identifiable-rule and labels are explicitly regime parameters over a fixed tagging mechanism.
- The data-flow map places minimization at every boundary, including the model prompt, tool calls, and logs.
- The residency map treats inference, storage, tool backends, and traces all as in-scope, defines crossing controls, and makes the allowed-region set a parameter.
- The retention schedule covers every data class with min/max/destruction, separates audit records from raw data, includes erasure, and makes schedules parameters.
- The regime-profile object is concrete, and the *same architecture* is shown under two different profiles with only parameters changing.

## Reflection

In `NOTES.md`:

1. Which data-flow boundary is most often forgotten in real systems, and what regulated data leaks through it when it is?
2. Retention pulls two ways (keep-for-audit vs. delete-for-privacy). Show a data class where they conflict and how separating the audit store from the raw store resolves it.
3. You switched the regime profile from one peer regime to another. List exactly what changed — and confirm nothing *structural* did.

## Stretch goals

- Design the **erasure** operation in detail: given a subject id, how do you enumerate and delete their data across agent memory (mod-303), vector store, logs, and run state — and what makes this only possible because of task-1 classification?
- Add a **residency-violation detector**: a runtime control that catches a data class reaching a disallowed region (e.g., a model endpoint failover to another region) and what it does.
- Reconcile your design with the convergence/divergence map from exercise-02: which of your controls were convergent (one mechanism, parameterized) vs. structural per regime?
