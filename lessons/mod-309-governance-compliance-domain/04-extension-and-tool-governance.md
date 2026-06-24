# Chapter 4 — Extension and Tool Governance

An agent's capabilities are exactly its tools. Every tool, plugin, extension, MCP server, or sub-agent the platform can invoke is an expansion of what the agent can *do* — and therefore of what must be governed. A platform that lets teams add tools freely, without a lifecycle around them, is an *ungoverned* platform no matter how clean its core: the next plugin can exfiltrate data, call a deprecated API, take an irreversible action, or quietly drift in quality until it is producing wrong answers in production.

This is the governance problem the horizontal frameworks of [Chapter 1](01-horizontal-governance-frameworks.md) flag directly — ISO/IEC 42001's "use of AI by third parties," the EU AI Act's supply-chain and oversight obligations, NIST RMF's third-party-risk Mapping. The architect's answer is a **lifecycle** every extension passes through and stays under: **evaluate → version → approve → monitor.** This chapter designs that lifecycle, for both third-party extensions (you did not write them, you trust them least) and internal tools (you wrote them, but they still drift and still need governing).

> **The extension surface is the real attack and drift surface.** The model is governed by your guardrails; the tools are governed only if you govern them. An agent with a perfectly safe core and an ungoverned tool registry is an unsafe agent. Treat the registry as a first-class governed system, not a config file.

## Why extensions need their own governance

A tool is not just code — it is a *capability grant* to a non-deterministic actor. Several properties make extensions uniquely dangerous and uniquely in need of lifecycle governance:

- **They expand the action surface.** Each tool is something the agent can now do to the outside world — read data, write data, spend money, send messages. The blast radius of the *agent* is the union of its tools' blast radii.
- **They are an egress and data-flow boundary.** Whatever context the agent passes to a tool leaves the agent's trust domain. Per the privacy primitive of [Chapter 2](02-regulated-domain-as-architecture.md), every tool is a place regulated data can leak.
- **Third-party ones are opaque and mutable.** A third-party plugin or MCP server is code you did not write, behind an interface that can change under you. "It was safe when we approved it" is not a durable property unless you pin and monitor it.
- **They drift.** Even an internal tool degrades — an upstream API changes, a prompt-shaped tool's quality regresses, a model the tool wraps is updated. Quality drift is silent and is exactly what monitoring exists to catch.
- **They compose.** Tools call other tools; sub-agents are tools. Governance must be transitive — approving a tool that itself calls three ungoverned tools governs nothing.

## The lifecycle: evaluate → version → approve → monitor

The governed lifecycle is a pipeline every extension enters before it can be called in production and stays inside for its whole life. Nothing reaches the agent's reachable tool set except through it.

```text
   submission
       │
       ▼
   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
   │  EVALUATE   │──▶│   VERSION   │──▶│   APPROVE   │──▶│   MONITOR   │
   │ security,   │   │ pin exact   │   │ named owner │   │ runtime     │
   │ permissions,│   │ version +   │   │ signs off;  │   │ behavior,   │
   │ quality,    │   │ immutable   │   │ scope &     │   │ drift,      │
   │ data flow   │   │ registry    │   │ conditions  │   │ abuse → ⟳   │
   └─────────────┘   └─────────────┘   └─────────────┘   └──────┬──────┘
                                                                │
                          revoke / quarantine ◀─────────────────┘
                          (and re-enter EVALUATE on change)
```

The loop is the point: monitoring feeds back into evaluation. A change in the extension, a drift signal, or an abuse pattern sends it back through the gate or pulls it from the registry entirely.

### Evaluate

Before an extension is eligible at all, it is assessed against an explicit rubric. The architect designs the rubric; it is not ad hoc:

- **Permissions and scope.** What does this tool need to access, and is that the *minimum*? A tool requesting broad data or write access it does not need is a finding. Scope it down to least privilege (ties to mod-306 tool scoping).
- **Data flow.** What context will the agent pass to it, where does that go, and does it cross a privacy/residency boundary from [Chapter 2](02-regulated-domain-as-architecture.md)? An extension that ships regulated data out of region fails here.
- **Security.** Injection surface, authentication, supply-chain provenance (who published it, is it signed, what are its dependencies). For third-party tools this is the heaviest gate.
- **Quality / correctness.** Does it do what it claims, reliably? For tools whose output the agent acts on, this needs an evaluation harness ([mod-304](../mod-304-evaluation-harnesses/README.md)) — a tool that is wrong 5% of the time is a governed risk, not a free capability.
- **Reversibility and consequence.** Does it take irreversible or high-consequence actions? If so it inherits the HITL-gate and idempotency discipline of [mod-308](../mod-308-deployment-durable-execution/README.md).

Third-party and internal extensions run the *same rubric*; the *evidence bar* differs. For an internal tool you can read the code and the tests; for a third-party one you lean on provenance, sandboxing, and tighter runtime monitoring because you cannot.

### Version

An approval is meaningless if the thing approved can change silently. So the platform **pins an exact version** and records it in an **immutable registry**:

- **Exact-version pinning.** The agent may call `tool@1.4.2`, never `tool@latest`. A new version is a *new submission* that re-enters Evaluate — approval does not transfer across versions.
- **An immutable, queryable registry** of what is approved, at which version, with which scope, approved by whom and when. This registry is itself an auditability artifact (Chapter 2): "which tools could this agent call on this date, and who approved them?" must be answerable.
- **Provenance and integrity.** A content hash or signature so the running tool is provably the approved one — preventing a swapped binary or a mutated MCP server from slipping past the gate.

This mirrors the in-flight versioning discipline of [mod-308](../mod-308-deployment-durable-execution/README.md): just as runs pin to a code version, agents pin to tool versions, and "approved" is always *version*-scoped.

### Approve

Evaluation produces evidence; approval is a **named human/role decision** to admit the extension to production, with conditions:

- **Named accountable approver.** A person/role signs off and is recorded — the accountability primitive of [Chapter 2](02-regulated-domain-as-architecture.md) applied to the tool registry. "The system approved it" is not approval.
- **Scoped admission.** Approval is bounded: which agents/fleets may use it, for which purposes, with which data classes, under which regime profiles. A tool approved for the internal-support fleet is *not* thereby approved for the regulated-data fleet.
- **Conditions and expiry.** Approvals can carry conditions (only with these guardrails active) and an expiry that forces periodic re-review, so "approved in 2024" does not mean "trusted forever."
- **Tiered rigor by risk.** A read-only internal utility and a third-party tool that can move money do not get the same approval ceremony. Tier the gate by blast radius, exactly as HITL gates are placed by consequence in [mod-308](../mod-308-deployment-durable-execution/README.md).

### Monitor

Approval is a point in time; behavior is continuous. Monitoring is what keeps a once-approved extension trustworthy and closes the loop:

- **Runtime behavior and policy enforcement.** Watch what the tool actually does at call time — arguments, data it touches, actions it takes — and enforce its approved scope at runtime, not just at admission. A tool reaching outside its scope is contained and flagged.
- **Quality-drift detection.** Track success/error rates and output quality against the baseline from Evaluate (via the eval harness and the observability metrics of [mod-305](../mod-305-observability-tracing/README.md)). Silent quality regression is the most common and least visible failure mode.
- **Abuse and anomaly detection.** Unusual call volumes, novel argument patterns, data-egress spikes — signals that the agent (or an attacker via prompt injection) is misusing the tool.
- **Revocation / quarantine path.** A fast way to pull an extension from the reachable set when it fails — a compromised third-party tool, a drifted internal one, a newly discovered vulnerability. Revocation must be *immediate* and must not require a full deploy; the registry is the control point.

## Third-party vs. internal: same lifecycle, different posture

Both kinds of extension go through the *same* four stages, but the architect tunes the posture:

| Aspect | Third-party extension | Internal tool |
| --- | --- | --- |
| Evaluate evidence | Provenance, signatures, sandboxed behavior tests (code is opaque) | Code review, unit/eval tests (code is yours) |
| Trust default | Least trust; assume mutable and possibly hostile | Higher trust, but *not* unlimited — internal tools drift too |
| Isolation | Strong sandboxing, network egress control, tight scope | Sandboxing still warranted for risky actions |
| Monitoring intensity | Heaviest — you cannot inspect the source | Lighter, but drift monitoring is still mandatory |
| Revocation readiness | Critical — supply-chain compromise is a live threat | Important — a bad internal release must be pullable fast |

The trap is exempting internal tools from the lifecycle ("we wrote it, it's fine"). Internal tools take the same actions and drift the same way; they belong in the same registry under the same monitoring, just with a lighter evidence bar at the Evaluate gate.

## Worked judgment

*A third-party "send-SMS" MCP server submitted for the customer-support fleet.* Evaluate flags: it can send messages to arbitrary numbers (irreversible, external comms), requests broad contact-list access (over-scoped), and is published by a vendor you cannot fully audit. The architecture's response: scope it to *approved templates and a verified recipient* only, pin the exact version with a signature, approve it for the support fleet *only* (not the regulated-data fleet) with a HITL gate on any non-template send, and monitor send-volume and recipient anomalies with a one-click revoke. The capability is admitted, but *governed* — bounded scope, pinned version, named approver, live monitoring, fast revocation.

*An internal "summarize-record" tool, version bump from 2.1 to 2.2.* Even though it is internal and low-action, the version bump re-enters Evaluate: re-run the eval harness to confirm summary quality did not regress, confirm it still touches only the approved data class, re-pin and re-register at 2.2. Skipping this because "it's just a minor bump" is how silent quality drift reaches production.

## Key takeaways

- An agent's capability *is* its tools, so the **extension surface is the real governance surface**. An ungoverned tool registry makes an otherwise-safe agent unsafe; treat the registry as a first-class governed system.
- Every extension — third-party *and* internal — passes through and stays inside the lifecycle **evaluate → version → approve → monitor**, and monitoring feeds back into evaluation. Nothing reaches the agent's reachable tools except through it.
- **Evaluate** against an explicit rubric (permissions, data flow, security, quality, consequence); **version**-pin exactly in an immutable registry with provenance; **approve** by a named owner with scoped, expiring, risk-tiered conditions; **monitor** runtime behavior and quality drift with an immediate revocation path.
- Third-party and internal extensions share the *same lifecycle* but differ in **posture** — evidence bar, isolation, monitoring intensity. Exempting internal tools is the classic failure; they take the same actions and drift the same way.
