# exercise-02: Plugin Lifecycle Governance

**Estimated effort:** 3 hours

## Objective

Design the end-to-end plugin governance lifecycle for an extensible agent platform — the evaluate → approve → version → monitor → revoke pipeline and its gates — and specify the manifest schema, the registry, the runtime enforcement, and the kill switch. By the end you will have a governance design that lets a platform stay extensible without becoming ungoverned.

## Background

This exercise covers material from:

- [Chapter 3 — Plugin and Extension Governance](../03-plugin-extension-governance.md)
- [Chapter 1 — Governance Frameworks for Agentic Systems](../01-governance-frameworks.md) (third-party / supplier controls)

A plugin is new code running with the agent's authority. Govern it as a supplier relationship and a life-cycle artifact at once.

## Scenario

Your platform (continue with TutorMesh from exercise-01, or any extensible agent platform) accepts plugins from three sources: **first-party** (your own team), **vetted partners**, and **open submission**. Three concrete plugins are in the queue:

- **`math-solver`** — open submission; takes a math expression, returns a worked solution; calls an external compute API.
- **`sis-writeback`** — first-party; writes a progress note into the student information system (a write to a system of record).
- **`translate-connector`** — vetted partner; translates agent output; sends text to a third-party translation API (an external egress of potentially student-derived text).

## Tasks

### 1. The manifest schema

- Define the machine-readable plugin manifest every submission must provide: declared tools, data classes touched, external endpoints, requested permission scope, provenance/trust tier, and version/hash.
- Fill the manifest out for all three plugins above. Apply **least privilege** — flag where a plugin requests more scope than its function needs.

### 2. The evaluation gate

- Specify what the evaluate step checks (security review, dependency/supply-chain scan, behavioral eval in a sandbox, data-handling review).
- For each of the three plugins, state the **risk rating** and **recommended scope** your evaluation would output. Justify the differences between the three trust tiers.

### 3. The approval gate and tiering

- Design tiered approval: which plugins auto-approve under policy, which need a named owner's sign-off, which need legal/privacy review. Map your three plugins onto the tiers.
- Specify what an approval record contains and how its conditions are enforced (by the registry, not by trust).

### 4. Versioning and the registry

- Specify the registry schema (identity, approved version + hash, scope grants, owner, risk rating, approval ref, status).
- Specify how signing/pinning works and what happens on a version bump (`translate-connector` ships v2.0 — walk the path).
- Call out the standing risk in `translate-connector` (a remote backend that can change without a version bump) and how you monitor it.

### 5. Monitoring and revocation

- Specify the runtime controls: scope enforcement, drift monitoring against the evaluation baseline, usage/data-access auditing.
- Design the revocation path: graceful deprecation vs. emergency kill switch, fleet-wide, fast, logged. Write the runbook step that revokes `sis-writeback` fleet-wide in an incident.

## Starter guidance

Use this manifest and registry outline. Treat it as a governance schema, not code.

```text
# Plugin manifest (submitted per plugin, per version)
plugin_id:        <stable id>
version:          <semver>
content_hash:     <hash of the artifact>
signature:        <signed by author key>
provenance:       first_party | vetted_partner | open
declared_tools:   [<tool name + read/write + target system>, ...]
data_classes:     [<e.g. student_education_record, none>, ...]
external_egress:  [<endpoint + what data leaves>, ...]
requested_scope:  <least-privilege scope statement>

# Registry entry (source of truth at runtime)
plugin_id | approved_version + hash | scope_grant | owner | risk_rating |
approval_ref | status (active|deprecated|revoked)

# Lifecycle gates (each is a gate or a control, not a courtesy)
EVALUATE → risk_rating + recommended_scope
APPROVE  → recorded named decision + conditions (tiered by risk)
VERSION  → signed, pinned, registry-tracked; re-evaluate on change
MONITOR  → runtime scope enforcement + drift vs. baseline + audit
REVOKE   → registry status flip → fleet-wide stop, logged
```

## Acceptance criteria

You can demonstrate that:

- A manifest schema exists and is filled for all three plugins, with at least one **least-privilege scope reduction** identified.
- Each plugin has a justified risk rating and recommended scope that reflects its trust tier and capability (the `sis-writeback` write and the `translate-connector` egress are treated as higher-risk than `math-solver`'s read).
- Approval is **tiered**, records conditions, and the conditions are enforced by the registry rather than by trust.
- Versioning is by signed hash, re-evaluation triggers on version change, and the remote-backend drift risk is explicitly monitored.
- A fleet-wide, fast, logged revocation path exists, with a written incident runbook step.

## Reflection

In `NOTES.md`:

1. Which plugin would you reject outright at evaluation, and what manifest field forced the decision?
2. Your monitoring flags `math-solver` returning wrong answers at v1.4 though it passed at v1.2. What in your design catches this, and what does it trigger?
3. Where would *over-governance* (gating low-risk plugins like everything else) push contributors to route around your process?

## Stretch goals

- Add multi-tenant isolation: how a plugin approved for district A is prevented from reaching district B's data, enforced at the sandbox.
- Design the SBOM/dependency story: a CVE lands in a transitive dependency shared by two plugins — how do you find every affected plugin from the registry?
- Specify the drill for the emergency kill switch and a metric for its mean-time-to-revoke.
