# exercise-01: Horizontal Framework Controls Mapping

**Estimated effort:** 3 hours

## Objective

Take an agentic system and produce a **sector-neutral controls map** against the two horizontal AI governance frameworks — **ISO/IEC 42001** and the **NIST AI RMF** — plus an **EU AI Act risk-tier classification**. For each relevant control/function, you name the place in the architecture that satisfies it. The deliverable is a clause-to-component map an auditor could walk and an engineer could implement against, produced *before* any sector-specific regulation enters the picture.

This is an **architecture** exercise. You will not implement anything; you produce a mapping table, a risk-tier justification, and an open-findings list. The whole point is to govern *horizontally* — do not reach for HIPAA, FERPA, PCI, or any statute here. Those belong in exercise-02.

## Background

This exercise covers material from:

- [Chapter 1 — Horizontal AI Governance Frameworks](../01-horizontal-governance-frameworks.md)

Assume your team can already build the agent, its tools, and its RAG pipeline (the L30 AI-Engineer-track skill). Your job is the layer above: showing that the system is *governable in general* by mapping framework requirements onto real components.

## The scenario

A reference agentic system, described sector-neutrally so no statute is implied:

> An **autonomous operations agent** for an enterprise. It ingests internal data, retrieves context via RAG, calls a set of tools (read internal records, write to a ticketing system, send notifications, call two external APIs), and runs largely autonomously. A small fraction of its actions are high-consequence (it can modify records and notify external parties). It runs as a fleet of workers, ships several deploys a week, and a few internal teams can add their own tools to it.

Treat the data as generically "sensitive enterprise data" — not any regulated category. The governance you derive here must hold regardless of which sector this is later sold into.

## Tasks

### 1. Classify the EU AI Act risk tier

- Using the tiers from [Chapter 1](../01-horizontal-governance-frameworks.md) (unacceptable / high / limited / minimal), classify this system and **justify** it from the system's autonomy, action surface, and affected parties — not from a sector.
- State which obligations the tier attaches (e.g., risk management, data governance, logging, human oversight, transparency) and note the transparency duty if users interact with it.

### 2. Map ISO/IEC 42001 controls to components

For the load-bearing Annex A control areas, fill a row mapping each to a real place in the architecture:

| 42001 control area | What it requires (your words) | Component / boundary / log / gate / owner that satisfies it | Status |
| --- | --- | --- | --- |

- Cover at least: AI system impact assessment, data for AI systems, AI lifecycle management, transparency to users, and use of AI by third parties (the tool surface).
- Mark each row **satisfied / partial / open** — an honest map has open findings.

### 3. Run the NIST AI RMF functions

- For **Govern**, **Map**, **Measure**, **Manage**, state what each means for *this* system. For Map, enumerate the top 4–6 risks of an autonomous, tool-calling, non-deterministic agent (include the action surface, not just output).
- For your top risks, name which component does the **Measuring** (eval harness, red-team, observability) and which does the **Managing** (guardrail, HITL gate, tool scoping). Reference the relevant prerequisite modules.

### 4. Produce the open-findings list

- Collect every control/function that resolved to "no component" or "partial." For each, state what component would close it. This list *is* the governance gap.

## Starter guidance

A mapping-row shape to adopt (do not just fill mechanically — justify):

```text
   control: "use of AI by third parties" (42001 Annex A)
   requires: governed lifecycle for any tool the agent can call
   component: the extension registry + evaluate→version→approve→monitor
             lifecycle (see Chapter 4)
   status: PARTIAL — registry exists, runtime monitoring not yet wired
```

The RMF functions as a checklist to instantiate for this agent:

```text
   GOVERN  : who is accountable for the fleet; stated risk tolerance
   MAP     : autonomy / tool-misuse / data-leakage / harmful-output /
             action-beyond-intent / prompt-injection risks
   MEASURE : eval harness (mod-304), red-team, observability (mod-305)
   MANAGE  : guardrails (mod-306), HITL gates (mod-308), tool scoping
```

You do **not** need any sector regulation here, and you do **not** design the extension lifecycle in detail (exercise-03) or the fleet specs (exercise-04) — reference them where a control points at them.

## Acceptance criteria

You can demonstrate that:

- The EU AI Act tier is classified and justified from the system's *properties*, not from a sector, and the attached obligations are named.
- The 42001 map covers the listed control areas, and **every** control resolves to a concrete component, boundary, log, gate, or owner — or is honestly marked open.
- The RMF functions are instantiated for this agent, the top risks include the *action* surface, and each top risk has a named Measure component and a named Manage component.
- The open-findings list is complete and each finding names the component that would close it.
- **No sector-specific statute (HIPAA, FERPA, PCI, etc.) appears** — the map is purely horizontal.

## Reflection

In `NOTES.md`:

1. Which 42001 control was hardest to point at a component for, and what does that say about a real gap in the architecture?
2. Your risk-tier classification drove a set of obligations *before any sector*. How much of the eventual architecture does the horizontal frame already mandate, in your estimate?
3. Pick one RMF "Manage" control you assigned to a guardrail. How would you *measure* that it is actually working, and where does that signal come from?

## Stretch goals

- Add a third framework lens: take one obligation the system inherits from its EU AI Act tier and show it is *also* a 42001 control *and* an RMF Manage action — i.e., one control satisfying three frameworks at once.
- Sketch the **certification readiness** view: if the org wanted a 42001 certificate, which of your open findings would block the audit, and in what order would you close them?
- Argue the *over-governance* failure mode: identify one control that, applied too zealously to this system, would add cost or friction without reducing real risk.
