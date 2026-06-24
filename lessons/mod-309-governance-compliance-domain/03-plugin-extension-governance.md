# Chapter 3 — Plugin and Extension Governance

An extensible agent platform is a feature and a liability in the same breath. Plugins — tools, connectors, skills, MCP servers, custom agents contributed by third parties or other teams — are what make the platform valuable and what make it ungovernable if you let them in unmanaged. Each plugin is **new code running with the agent's authority, often touching regulated data, sometimes changing the system itself**. From a governance standpoint a plugin is a supplier relationship and a piece of the AI system life cycle simultaneously (recall the ISO/IEC 42001 Annex A third-party and life-cycle controls from Chapter 1). This chapter designs the lifecycle that keeps an extensible platform a governed one.

## What a plugin actually is, from a risk lens

Before designing controls, name the risk. A plugin can:

- **Exfiltrate data.** A connector with read access to student or health records can leak it to an external endpoint.
- **Take unauthorized actions.** A tool with a `write` or `delete` capability extends the agent's reach into systems that were never risk-assessed for autonomous action.
- **Degrade quality silently.** A plugin that returns plausible-but-wrong data poisons the agent's reasoning — a correctness and safety regression, not a crash.
- **Change the system.** A plugin that can edit configuration, install other plugins, or modify prompts can alter the platform's behavior outside any review — the unauthorized-system-change risk this module's objectives call out by name.
- **Drift.** A plugin that was safe at v1.2 can become unsafe at v1.3, or its remote backend can change behavior with no version bump at all.

Governance must address all five, and it must address them *before* the plugin runs, *while* it runs, and *after* something goes wrong.

## The lifecycle: evaluate → approve → version → monitor → revoke

```text
  submit ──▶ EVALUATE ──▶ APPROVE ──▶ VERSION ──▶ MONITOR ──▶ REVOKE
              │ (gate)      │ (gate)    (registry)  (runtime)   (kill)
              ▼            ▼                          │            │
           reject      conditions                     └─ feedback ─┘
                                                       (re-evaluate on drift)
```

Each arrow is a gate or a control, not a courtesy. Walk them.

### 1. Evaluate (the intake gate)

No plugin enters the platform without passing a defined evaluation. The evaluation gate inspects:

- **Declared capabilities and scopes.** The plugin must declare, in a machine-readable **manifest**, every tool it exposes, every data class it touches, every external endpoint it calls, and the permission scope it requests (read-only? write? which records?). A plugin that requests broad scopes for a narrow function is rejected or scoped down — least privilege is the default.
- **Provenance and identity.** Who authored it? First-party, a vetted partner, or anonymous? Provenance sets the trust tier and the depth of review.
- **Security review.** Static analysis, dependency/supply-chain scan, secret scanning, and a review of the network egress it performs. For higher trust tiers, dynamic analysis in a sandbox.
- **Behavioral evaluation.** Run it against an eval suite (mod-304) in a sandbox: does it do what it claims, refuse what it should, and stay within its declared scope? Capture a baseline of its behavior — you will diff against this later.
- **Data-handling review.** If it touches regulated data, it inherits the obligations from Chapter 2: minimization, region pinning, a data-processing agreement, deletion support. A plugin that cannot honor a deletion request does not get access to deletable data.

The output of evaluation is a **risk rating** and a **recommended scope**, not just pass/fail.

### 2. Approve (the authorization gate)

Approval is a named, recorded human decision — the accountability link from Chapter 2.

- **Tiered approval by risk.** A read-only, first-party plugin might be auto-approved under policy. A third-party plugin with write access to student records requires an accountable owner's explicit sign-off and possibly legal/privacy review. Match approval rigor to the risk rating; do not gate everything identically (over-governance trains teams to route around you).
- **Conditions of approval are recorded.** "Approved for read-only access to tutoring-tagged records, in the US region, expires in 12 months, owner = J. Rivera." Conditions are enforced by the registry, not by trust.
- **The approval is the authority.** Per Chapter 2, every later tool grant traces to this approval record. No approval, no grant, no execution.

### 3. Version (the registry and supply-chain control)

Approval is of a *specific version*, not a name.

- **Signed, immutable, pinned versions.** The platform runs a specific plugin version identified by a content hash, with a **signature** verified at load time. An unsigned or hash-mismatched plugin fails closed. This defeats the "safe at v1.2, malicious at v1.3" and tampering risks.
- **A registry as the source of truth.** The plugin registry records: plugin identity, approved version(s) and their hashes, scope grants, owner, risk rating, approval record, and status (active / deprecated / revoked). The agent runtime consults the registry; it does not load plugins from arbitrary sources.
- **Pinned dependencies and SBOM.** Capture a software bill of materials so a downstream CVE in a transitive dependency is traceable to every plugin that ships it.
- **Re-evaluation on change.** A new version re-enters at Evaluate. A plugin whose *remote backend* can change without a version bump is a standing risk — flag these, monitor them harder, and prefer plugins whose behavior is pinnable.

### 4. Monitor (the runtime control)

Approval is a point-in-time judgment; behavior is continuous. Monitoring closes the gap.

- **Runtime scope enforcement.** The sandbox/permission layer (mod-306) enforces the approved scope at call time. A plugin attempting an out-of-scope action is blocked *and the attempt is logged as a signal* — an in-scope plugin should never try to exceed scope; one that does has drifted or been compromised.
- **Quality and behavior monitoring.** Track per-plugin error rates, latency, and output-quality signals against the evaluation baseline. A plugin whose behavior drifts from baseline triggers re-evaluation. This catches silent quality degradation, which no point-in-time gate can.
- **Usage and data-access auditing.** Every plugin invocation lands in the audit log (Chapter 2): which plugin, version, scope, data touched, by which agent. This is both compliance evidence and the forensic trail for incidents.
- **Anomaly detection.** Sudden changes in egress volume, new endpoints contacted, or spikes in privileged-action attempts are alerts.

### 5. Revoke (the kill switch)

Governance you cannot reverse is governance you do not have.

- **Per-plugin, fleet-wide revocation.** Flipping a plugin to `revoked` in the registry must stop every agent in the fleet from loading or calling it — fast, and without a full redeploy. This is the containment control for a compromised or misbehaving plugin.
- **Graceful vs. emergency revocation.** *Deprecation* gives downstream agents a migration window; *emergency revocation* is immediate and accepts breakage to stop harm. Both must exist; the emergency path must be drilled (an undrilled kill switch is a hope, not a control).
- **Revocation is an event.** It is logged, attributed, and feeds an incident or change record, so the "what happened and why" trail stays intact.

## Multi-tenant and marketplace considerations

If your platform hosts plugins from many parties for many customers, layer two more controls:

- **Tenant isolation.** A plugin approved for tenant A must not see tenant B's data; scope grants are tenant-scoped, and the sandbox enforces tenant boundaries.
- **A statement of applicability per plugin.** For regulated customers, you may need to show *per plugin* which controls apply and what data it can reach — the same evidence discipline as the framework mapping in Chapter 1, at plugin granularity.

## Anti-patterns

- **Trust by allowlist alone.** "It's on the list" without signing, scoping, and monitoring is a name, not a control.
- **Approve once, run forever.** No re-evaluation on version change and no drift monitoring lets a once-safe plugin rot into a liability.
- **Scope by convention.** "Plugins are expected not to write outside their area" with no runtime enforcement is a comment, not a boundary.
- **No revocation path.** If pulling a bad plugin requires a redeploy, you do not have containment.

## Key takeaways

- A plugin is new code running with the agent's authority — capable of exfiltration, unauthorized action, silent quality degradation, and unauthorized system change. Govern it as a supplier relationship and a life-cycle artifact at once.
- The lifecycle is **evaluate → approve → version → monitor → revoke**, and each step is a gate or control: a machine-readable manifest with least-privilege scopes, a tiered and recorded approval, signed/pinned/registry-tracked versions, runtime scope enforcement plus drift monitoring, and a fast fleet-wide kill switch.
- Approval is of a *specific signed version*, traces to a named owner, and is re-triggered on any change; behavior is monitored continuously against an evaluation baseline because point-in-time gates miss drift.
- Match gate rigor to risk, enforce scope at runtime rather than by convention, and drill the emergency revocation path — an undrilled kill switch is not a control.
