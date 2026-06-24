# Extensible Agent Platform (Platform-Neutral)

**Duration**: 25 hours | **Difficulty**: High | **Project ID**: project-303-extensible-agent-platform

## Overview

Real agentic value comes from letting agents securely reach into an organization's own toolchains and proprietary data to do real work. This project asks you to design an **extensible agent platform**: a host-plus-extension model where agents, tools, hooks, and external connectors can be added safely over time, under strong auth and isolation, with a governance lifecycle that keeps extensions trustworthy.

The defining constraint is **platform neutrality**. You will design against a generic host/extension model and justify it against at least one concrete alternative platform — treating Claude Code, Cursor, Copilot, LangGraph Platform, and a fully custom host as **peers**, not as the assumed answer. You must address portability and lock-in head-on: what is portable across hosts, what is host-specific, and how a team would migrate.

### The worked domain is your choice

Pick one domain to make the design concrete. It does **not** have to be developer tooling:

- A software-delivery (SDLC) workflow — code review, dependency triage, release notes.
- A non-developer internal workflow — support-ticket triage, sales-ops research, content operations, internal knowledge Q&A, HR/finance request handling.

Do not frame the platform as a single company's internal AI-for-SDLC product. It is a general extensible platform; the worked domain is one instantiation chosen to exercise the design.

## Learning Objectives

By completing this project, you will:

1. Design a host/extension architecture (agents, tools, hooks, MCP-style connectors) that is open to extension and closed to abuse.
2. Build a secure auth and sandboxing model spanning identity, authorization, token lifecycle, and execution isolation.
3. Defend the platform against prompt injection and untrusted-tool-output attacks.
4. Define an extension governance lifecycle from proposal through deprecation.
5. Integrate real toolchains (VCS, issue tracker, knowledge base, CI/CD) through a uniform extension contract.
6. Reason explicitly about portability and lock-in by comparing your generic model to a named alternative platform.

## Project Scenario

### Context

You are the architect of an extensible agent platform that multiple teams will extend independently. Some extensions are first-party; some are written by other teams; eventually some may be third-party. Extensions need access to sensitive systems and proprietary data, which means a careless extension — or a malicious tool output — must not be able to exfiltrate data, escalate privilege, or take destructive actions unchecked.

Leadership is wary of betting the platform on a single vendor's host. You must justify your host/extension model against at least one alternative and show how a team could move workloads if the chosen host changed. "We will just use X" is not an acceptable architecture; the reasoning for X over its peers, and the lock-in cost of that choice, is the work.

### Your mission

Design the extensible platform, prove the extension contract on real toolchain integrations, specify the security and governance models, and deliver an adoption/DX plan that would get other teams building extensions safely.

## Requirements

### Functional requirements

1. **FR-1 — Extension architecture**: A host/extension model defining the four extension types — agents, tools, hooks, and external connectors (MCP-style) — with a uniform contract: declared capabilities, declared permissions, inputs/outputs, and lifecycle. The host must be able to load, isolate, and revoke extensions.
2. **FR-2 — Secure auth and sandboxing**: An identity and authorization model (OAuth for delegated access; RBAC and/or ABAC for what an extension may do) with a token lifecycle (issuance, scoping, rotation, revocation, least privilege), plus execution isolation for both local and remote extension execution.
3. **FR-3 — Prompt-injection and untrusted-output defenses**: Defenses against prompt injection and against malicious or compromised tool output, including a trust boundary between model-generated intent and privileged action, and an action-confirmation/authorization gate for high-impact operations.
4. **FR-4 — Extension governance lifecycle**: A lifecycle from proposal -> review -> approval -> publish -> versioning -> deprecation -> removal, including how permissions are reviewed, how a bad extension is pulled, and how breaking changes are handled.
5. **FR-5 — Toolchain integrations**: At least three working integrations across these categories — version control, issue tracker, knowledge base, CI/CD — implemented through the FR-1 extension contract (not bespoke one-offs).
6. **FR-6 — Platform comparison and portability**: A reasoned comparison of your generic host/extension model against at least one named alternative (Claude Code, Cursor, Copilot, LangGraph Platform, or custom), with an explicit portability and lock-in analysis: what is portable, what is host-specific, and the migration path.
7. **FR-7 — Adoption and DX plan**: How another team designs, tests, and ships an extension; the developer experience (scaffolding, local testing, docs); and the path from prototype to approved, published extension.

### Non-functional requirements

1. **Security**: Default-deny permissions; no extension gets ambient authority; secrets never reach extension code in plaintext beyond the minimum scope.
2. **Isolation**: A misbehaving extension cannot read another extension's secrets, exceed its declared permissions, or compromise the host.
3. **Portability**: The extension contract is expressible against more than one host with a bounded, documented amount of host-specific glue.
4. **Operability and auditability**: Every extension action is attributable and logged; permissions and versions of every loaded extension are inspectable.
5. **Developer experience**: A new extension can go from idea to local test quickly, with clear contracts and feedback.

### Constraints

- Platform-neutral by design — no single host is the assumed answer.
- At least one alternative platform must be evaluated as a genuine peer.
- The worked domain may be SDLC or a non-developer internal workflow — your choice, stated up front.
- Integrations must go through the extension contract, demonstrating extensibility rather than hard-coding.

## Deliverables

Suggested layout:

```text
project-303-extensible-agent-platform/
├── README.md                    # This file
├── architecture/                # Host/extension model + diagrams
├── extension-contract/          # The uniform contract (schema + examples)
├── security/                    # Auth, sandboxing, token lifecycle, injection defenses
├── governance/                  # Extension lifecycle (proposal -> removal)
├── integrations/                # >=3 toolchain integrations via the contract
├── portability/                 # Platform comparison + lock-in analysis
├── adoption-dx/                 # Adoption + developer-experience plan
└── adrs/                        # Key decisions (>=8)
```

1. **Extension architecture + diagrams** — Host/extension model with the four extension types, the load/isolate/revoke flow, and where the trust boundary sits. Include a sequence diagram of an extension invoking a privileged action through the authorization gate.
2. **Extension contract** — The uniform contract (a schema plus at least two worked examples): declared capabilities, requested permissions, I/O, and lifecycle hooks.
3. **Security model** — OAuth delegation, RBAC/ABAC authorization, token lifecycle (issue/scope/rotate/revoke), local vs. remote execution isolation, and the prompt-injection/untrusted-output defenses with the action-confirmation gate.
4. **Governance lifecycle** — The full proposal-to-removal lifecycle, including permission review, version/deprecation policy, and the kill-switch for a bad extension.
5. **Toolchain integrations** — At least three integrations (VCS / issue tracker / knowledge base / CI/CD) built through the contract, runnable or clearly specified with stubs.
6. **Portability and lock-in analysis** — The comparison against at least one named alternative platform, with the portable/host-specific split and a concrete migration path.
7. **Adoption and DX plan** — The extension developer journey, scaffolding and local-test story, docs plan, and the prototype-to-published path.
8. **ADR set (>=8)** — Host model, isolation mechanism, auth model, injection-defense placement, governance gates, the alternative-platform choice, integration approach, and worked-domain choice.

## Effort breakdown (25 hours)

| Phase | Focus | Hours | Primary deliverable |
|-------|-------|-------|---------------------|
| 1 | Worked-domain choice, host/extension model, extension contract | 5 | Architecture + extension contract |
| 2 | Security model: auth, RBAC/ABAC, token lifecycle, isolation | 6 | Security spec |
| 3 | Prompt-injection / untrusted-output defenses + authorization gate | 3 | Injection-defense spec |
| 4 | Toolchain integrations through the contract (>=3) | 5 | Integrations |
| 5 | Extension governance lifecycle | 2 | Governance spec |
| 6 | Platform comparison + portability/lock-in analysis | 2 | Portability analysis |
| 7 | Adoption/DX plan + package review | 2 | Adoption/DX plan |

## Acceptance criteria

- [ ] The host/extension model defines agents, tools, hooks, and connectors under one uniform contract with load/isolate/revoke.
- [ ] Auth model uses OAuth for delegation and RBAC and/or ABAC for authorization; permissions are default-deny.
- [ ] Token lifecycle covers issuance, scoping, rotation, and revocation with least privilege.
- [ ] Execution isolation is specified for both local and remote extension execution.
- [ ] Prompt-injection and untrusted-tool-output defenses exist, with a trust boundary and an action-confirmation gate for high-impact operations.
- [ ] An extension governance lifecycle runs from proposal through deprecation and removal, including a kill-switch.
- [ ] At least three toolchain integrations are implemented through the extension contract, not hard-coded.
- [ ] At least one named alternative platform is evaluated as a peer, with an explicit portability and lock-in analysis and a migration path.
- [ ] The adoption/DX plan describes the extension developer journey from prototype to published.
- [ ] The platform is platform-neutral and the worked domain (SDLC or non-developer workflow) is stated and not framed as one company's internal tool.
- [ ] At least 8 ADRs document the key decisions.

### Rubric

| Dimension | Weight | What strong work looks like |
|-----------|--------|------------------------------|
| Extensibility and contract design | 25% | One clean contract covers all four extension types; integrations prove it; adding an extension is principled, not bespoke. |
| Security and sandboxing | 25% | Default-deny auth, sound token lifecycle, real isolation; a misbehaving extension is contained. |
| Injection and untrusted-output defense | 15% | A clear trust boundary and authorization gate; model-generated intent cannot directly drive privileged action. |
| Governance lifecycle | 10% | Proposal-to-removal is operable, with permission review, versioning, and a working kill-switch. |
| Portability and lock-in | 15% | Honest peer comparison; portable vs. host-specific is explicit; the migration path is concrete. |
| Adoption and DX | 10% | A new team could realistically build and ship a safe extension from the plan. |

## Stretch goals

- **Second host binding**: Express the extension contract against a second host and report exactly how much glue was host-specific.
- **Capability attestation**: Add signing/attestation so the host can verify an extension's declared permissions before loading.
- **Injection red-team**: Plant a malicious instruction in tool output and demonstrate the authorization gate blocks the privileged action.
- **Permission diffing**: On extension upgrade, surface and require re-approval of any permission expansion.
- **Marketplace governance**: Extend the lifecycle to third-party extensions with a vetting and revocation model.

## Submission guidelines

1. State your worked domain and your chosen alternative platform up front in ADR-001 and ADR-002.
2. Ensure integrations actually flow through the extension contract — that is the heart of the grade.
3. Keep the portability/lock-in analysis honest; name what you would lose by switching hosts.
4. Self-assess against the rubric and record open security questions in ADRs.

---

**Ready to start?** Begin with Phase 1: choose your worked domain and draft the extension contract.
