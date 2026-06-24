# exercise-03: Extension and Tool Governance Lifecycle

**Estimated effort:** 3 hours

## Objective

Design the **evaluate → version → approve → monitor** lifecycle for an agent platform's extension surface, covering both **third-party** and **internal** tools. The deliverable is an evaluation rubric, a versioning-and-registry policy, an approval contract, a monitoring-and-revocation design, and a worked walk-through of one third-party and one internal extension through the whole pipeline.

This is an **architecture** exercise. You design the *governance process and its gates*, not a working plugin loader.

## Background

This exercise covers material from:

- [Chapter 4 — Extension and Tool Governance](../04-extension-and-tool-governance.md)
- [Chapter 2 — Regulated Domain as Architecture](../02-regulated-domain-as-architecture.md) — every tool is a data-flow/egress boundary.

## The scenario

> You operate an extensible agent platform. Internal teams add their own tools, and the product also accepts **third-party tools / MCP servers**. Today, a tool is added by merging a PR that registers it — there is no evaluation, no version pinning, no monitoring, and no way to revoke a tool short of a redeploy. A third-party "send-message" tool was recently added that can message arbitrary recipients, and an internal "summarize-record" tool silently regressed in quality after an upstream model change.

Design the lifecycle that fixes this.

## Tasks

### 1. Design the evaluation rubric

- Specify the rubric every extension is assessed against before it is eligible, per [Chapter 4](../04-extension-and-tool-governance.md): permissions/scope (least privilege), data flow (privacy/residency boundary crossing), security/provenance, quality/correctness, and reversibility/consequence.
- For each rubric dimension, state the **evidence** required and how it differs for third-party (opaque) vs. internal (code you own) extensions.

### 2. Write the versioning and registry policy

- Specify exact-version pinning (the agent calls `tool@x.y.z`, never `@latest`) and that a new version **re-enters Evaluate**.
- Design the **immutable registry** record: what is stored (version, scope, approver, timestamp, provenance hash) and how it doubles as an auditability artifact ("which tools could this agent call on date D?").

### 3. Write the approval contract

- Specify approval as a **named-owner** decision with: scoped admission (which fleets/purposes/data classes/regime profiles), conditions, expiry/re-review, and **risk-tiered rigor** (a read-only utility ≠ a money-moving third-party tool).

| Field | Definition |
| --- | --- |
| allowed scope | which agents/fleets/purposes/data classes |
| approver | named role, recorded |
| conditions | required guardrails / regime profile |
| expiry | re-review deadline |
| risk tier | rigor level of the gate |

### 4. Design monitoring and revocation

- Specify runtime scope enforcement (the tool is contained to its approved scope at *call* time), quality-drift detection (against the Evaluate baseline, via mod-304/mod-305), and abuse/anomaly detection.
- Design the **revocation/quarantine path**: how a tool is pulled from the reachable set *immediately*, without a full deploy. Identify the control point (the registry).

### 5. Walk two extensions through the pipeline

- **Third-party** "send-message" tool: run it through evaluate → version → approve → monitor. Show the scope reduction, the version pin, the bounded approval, and the monitoring + revoke design.
- **Internal** "summarize-record" tool, version bump after the model change that caused the regression: show why the bump re-enters Evaluate and how the eval harness catches the drift *before* approval.

## Starter guidance

The lifecycle to instantiate end-to-end:

```text
   submission ─▶ EVALUATE ─▶ VERSION ─▶ APPROVE ─▶ MONITOR ─┐
                  ▲ rubric    pin+registry  named owner  runtime+drift
                  └──────────── re-enter on change / drift ─┘
                  revoke / quarantine ◀── fast path (registry control point)
```

An evaluation-row shape to adopt:

```text
   dimension: data flow
   third-party: must declare what context it receives + where it sends;
                fails if regulated data crosses a residency boundary
   internal:    same rule, verified by reading the code + tests
```

You do **not** need the full fleet-governance specs here (exercise-04) — but note that the registry record feeds the lineage spec there.

## Acceptance criteria

You can demonstrate that:

- The evaluation rubric covers all five dimensions, with the evidence bar differentiated for third-party vs. internal extensions.
- The versioning policy pins exact versions, forces re-evaluation on change, and defines an immutable registry that doubles as an audit artifact.
- The approval contract is a named-owner, scoped, expiring, risk-tiered decision — not an automatic merge.
- Monitoring enforces scope at runtime, detects quality drift against a baseline, and the revocation path is immediate and does not require a redeploy.
- Both worked extensions traverse the full pipeline, and the internal one shows drift caught *before* re-approval.

## Reflection

In `NOTES.md`:

1. The platform team objects that the lifecycle slows down internal tool additions. How do you keep internal tools governed without killing velocity — and why is fully exempting them the wrong answer?
2. A third-party tool you approved at v1.2 ships v1.3 with a "minor" change. Walk through what your policy does and why approval does not transfer.
3. Transitivity: a tool you approved itself calls three other tools. What does your lifecycle require of *those* tools, and what breaks if you ignore it?

## Stretch goals

- Design **sandboxing tiers** keyed to the risk tier — what isolation (network egress control, scoped credentials, resource limits) each tier mandates.
- Add a **supply-chain** check to Evaluate for third-party tools: signatures, dependency provenance, and what you do when a dependency later gets a CVE.
- Connect the registry to the **deploy versioning** of mod-308: when the agent code version changes, how do you ensure the tool-version pins still hold for in-flight runs?
