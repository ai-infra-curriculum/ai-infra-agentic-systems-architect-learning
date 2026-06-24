# Chapter 3 — Multi-Regime Mapping

A real platform rarely faces one regulation. A single agent product gets sold into a hospital network, then a bank, then a government agency, then a school district — and now *one architecture* must satisfy HIPAA, GLBA/SOX/PCI, public-sector records and procurement rules, and FERPA/COPPA at the same time. The naive response is to fork: a healthcare build, a finance build, a gov build, an edtech build. That path is a maintenance catastrophe and a governance one — four codebases drifting apart, each its own audit surface.

The architect's response is the opposite: **map the one architecture against all the regimes at once, and find where their controls converge versus diverge.** [Chapter 2](02-regulated-domain-as-architecture.md) gave you the tool for this — the five primitives are statute-independent, so most controls *converge* across regimes and can be built once. The work of this chapter is to (1) treat the sectors as **equal peers**, with none as the default frame, (2) identify the **convergent core** you build once, and (3) isolate the **divergent deltas** you parameterize. The deliverable is a convergence/divergence map that lets one platform serve every regime.

> **Peers, not a primary plus exceptions.** It is tempting to pick one sector as "the main case" and treat the others as variations. Resist it — that choice silently biases the architecture toward one regime's assumptions. Healthcare, finance, public sector, and edtech are *balanced peers* here. We rotate which one leads each example on purpose.

## The four peer regimes

Each regime is summarized by the same five primitives, so they can be compared on equal footing. The point is not to memorize statutes — it is to see that they are *the same dials at different settings.*

| Sector | Representative regimes | Data subject | Primitive emphasis (illustrative) |
| --- | --- | --- | --- |
| **Healthcare** | HIPAA (+ regional health-data law) | Patient | Strong privacy (PHI), long retention, residency in some jurisdictions, clinician accountability |
| **Finance** | GLBA, SOX, PCI DSS | Customer / account holder | Privacy (financial data) + audit-record integrity (SOX) + card-data handling (PCI) + long retention |
| **Public sector** | Records-management, procurement, transparency, sovereignty law | Citizen | Residency/sovereignty, strong auditability and transparency, public-records retention, accountability of officials |
| **Edtech** | FERPA, COPPA (+ regional child-data law) | Student (often a minor) | Privacy (education records), consent for minors, retention limits, parental/institutional accountability |

Read across the rows and the structure is identical: every regime is about *who may see the data, where it lives, what is recorded, who is accountable, and how long it is kept.* That is the convergence. The differences are in the **thresholds and the extras** (PCI's card-data specifics, COPPA's verifiable parental consent, SOX's control-attestation, public-sector sovereignty) — the divergence.

## Convergence: the controls you build once

Because the primitives are shared, a large **convergent core** of controls satisfies every regime simultaneously, just tuned by parameters. Build these once:

- **Data classification and tagging at ingest.** Every regime needs to know what class of sensitive data it holds. The *labels* differ (PHI, cardholder data, education record, PII), but the *mechanism* is one classifier and tag schema. Build once; configure the label set per regime.
- **Minimization before model/tool/log exposure.** Every regime wants the minimum-necessary identifiable data exposed. One masking/tokenization boundary serves all; the *definition of identifiable* is the parameter.
- **Immutable, queryable audit log of decisions and actions.** Healthcare, finance, public sector, and edtech all demand provable records. One tamper-evident decision log (on the durable history of [mod-308](../mod-308-deployment-durable-execution/README.md)) serves all; the *retention window* and *required fields* are parameters.
- **Human-accountability gates by consequence.** Every regime wants a named human accountable for high-consequence actions. One HITL-gate framework serves all; *which actions are gated and who may decide* is the parameter.
- **Per-class retention lifecycle.** Every regime sets minimum/maximum lifetimes and destruction rules. One lifecycle engine serves all; the *schedule* is the parameter.
- **Access policy scoped by purpose and role.** Every regime restricts who may use data for what. One policy engine serves all; the *roles and purposes* are parameters.

This convergent core is most of the architecture. The strategic insight: **the bulk of multi-regime compliance is parameterization of a shared platform, not per-regime engineering.** If your architecture forces a fork to add a regime, the convergent core was not factored out.

## Divergence: the deltas you parameterize (or build per regime)

What is left is genuinely regime-specific. Divergence comes in two grades, and telling them apart is the architect's core judgment here.

**Parameter-level divergence** — same control, different setting. Handle by configuration:

- *Residency region set* — healthcare-in-jurisdiction-X vs. public-sector-sovereign-cloud vs. finance-cross-border-allowed: same region-pinning mechanism, different allowed-region list.
- *Retention schedules* — long for medical and financial records, often shorter and consent-bound for student data: same lifecycle engine, different schedule.
- *"Identifiable" threshold* — what must be masked differs by regime: same masking boundary, different rule set.
- *Gate authority* — a clinician approves in healthcare, a compliance officer in finance, a records officer in public sector, a parent/guardian or educator in edtech: same gate framework, different authority policy.

**Structural divergence** — a regime needs a control the others do not, requiring an actual new component:

- *Verifiable consent for minors* (edtech/COPPA-style) — a consent-capture and verification component with no analog in a finance-only build. Build it; activate it only for regimes that require it.
- *Card-data handling and scope reduction* (finance/PCI) — tokenization and network-segmentation requirements specific enough to be their own component, ideally keeping cardholder data out of the agent's reach entirely.
- *Control-attestation and change-management evidence* (finance/SOX) — a structured attestation artifact tying controls to named owners and tested operation.
- *Public-records transparency / disclosability* — a component that can produce records on request and distinguish disclosable from exempt content.

The design rule: **parameter-level divergence is configuration on the convergent core; structural divergence is a pluggable component, activated per regime.** Either way, you do not fork the platform — you compose it.

```text
   ┌──────────────────────────────────────────────────────────────┐
   │  CONVERGENT CORE (build once, satisfies all regimes)          │
   │  classify · minimize · audit-log · accountability gates ·     │
   │  retention lifecycle · purpose/role access policy             │
   └───────────────┬──────────────────────────────────────────────┘
                   │  configured by REGIME PROFILE (parameters)
                   │  region set · retention schedule · identifiable
                   │  rules · gate authority · required log fields
                   ▼
   ┌──────────────────────────────────────────────────────────────┐
   │  PLUGGABLE STRUCTURAL DELTAS (activated per regime)           │
   │  [minor consent] [card-data scope] [SOX attestation]         │
   │  [public-records disclosure] ...                              │
   └──────────────────────────────────────────────────────────────┘
```

## When regimes stack or conflict

Real deployments often sit under *several* regimes at once, and the rules can pull in different directions. The architect must resolve this explicitly:

- **Stacking (most common).** A system handles both health and payment data, so HIPAA *and* PCI apply to different data classes in the same platform. Resolution: classification routes each data class to the controls its regime demands — the convergent core already supports per-class handling.
- **Most-restrictive-wins (the usual conflict rule).** When two regimes set different thresholds on the same data, the architecture generally applies the stricter (shorter max-retention, broader masking, tighter residency). Encode this as a *resolution policy* over the regime profiles, not as an ad-hoc per-feature decision.
- **Genuine conflict (rare but real).** Occasionally one regime *requires* what another *forbids* — e.g., a retention-minimum from one regime against a deletion-right from another for the same datum. This is not an architecture problem to solve silently; it is a governance escalation. The architecture's job is to *surface* the conflict (flag the datum as under conflicting regimes) and route it to a human/legal decision, then enforce whatever resolution comes back.

## The architect's deliverable: a convergence/divergence map

The artifact that ties this chapter together is a table mapping the one architecture against the peer regimes, marking each control as convergent (build once) or divergent (parameter vs. structural). Schematically:

```text
   control            │ healthcare │ finance │ public │ edtech │ verdict
   ───────────────────┼────────────┼─────────┼────────┼────────┼──────────────
   data classification│  PHI       │ card/PII│ PII    │ edu rec│ CONVERGE (param labels)
   minimization       │  ✓         │ ✓       │ ✓      │ ✓      │ CONVERGE (param threshold)
   immutable audit log│  ✓ long    │ ✓ SOX   │ ✓ pub  │ ✓      │ CONVERGE (param retention)
   accountability gate│ clinician  │ officer │ records│ educator│ CONVERGE (param authority)
   residency          │ in-region  │ varies  │ sovrgn │ varies │ CONVERGE (param region set)
   minor consent      │  —         │ —       │ —      │ ✓ COPPA│ DIVERGE (structural)
   card-data scope    │  —         │ ✓ PCI   │ —      │ —      │ DIVERGE (structural)
   records disclosure │  —         │ —       │ ✓      │ —      │ DIVERGE (structural)
```

A map like this *is* the multi-regime architecture decision: it shows what you build once, what you parameterize, and the small set of structural components you activate per regime — and it proves you did not fork the platform four ways.

## Worked judgment

*Lead with public sector this time.* A case-management agent for a government agency must satisfy sovereignty (residency in a national/sovereign cloud), strong public-records auditability, and accountability of named officials. Mapped against the convergent core, *every one of those is an existing control with a different parameter*: residency → sovereign-region set; auditability → existing immutable log with a public-records retention schedule and a disclosure component bolted on; accountability → existing gate framework with official-role authority. The only *new* build is the public-records **disclosure** component (structural divergence). The same platform that ran under HIPAA and PCI runs here by swapping the regime profile and activating one component — which is exactly the win the convergence analysis buys.

## Key takeaways

- A production platform faces **many regimes at once**; forking per regime is a governance and maintenance failure. The architect maps one architecture against all regimes and separates convergence from divergence.
- Treat sectors as **equal peers** — healthcare, finance, public sector, edtech — none the default. Because the five primitives are statute-independent, **most controls converge** and are built once, tuned by a per-regime profile.
- **Divergence has two grades:** parameter-level (same control, different setting → configuration) and structural (a control one regime needs and others do not → a pluggable component activated per regime). You compose the platform; you do not fork it.
- When regimes **stack or conflict**, resolve with explicit policy — per-class routing, most-restrictive-wins for overlaps, and *escalation* (not silent choice) for genuine conflicts. The deliverable is a **convergence/divergence map** that proves the platform serves every regime without a fork.
