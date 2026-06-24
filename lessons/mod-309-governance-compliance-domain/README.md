# mod-309-governance-compliance-domain: Governance, Compliance & Domain Constraints

**Estimated effort:** 15 hours

By the time an agentic platform reaches production, the hard problems are no longer "can the agent do it?" — they are "are we *allowed* to ship it, can we *prove* we governed it, and who is *accountable* when it goes wrong?" Those are architecture problems, not afterthoughts. A fleet of autonomous agents that takes actions, calls tools, ingests regulated data, and produces output for vulnerable end-users is a regulated system whether or not anyone wrote it down. This module is where you, as the architect, make the governance explicit: you map a recognized management-system standard onto your platform, translate regulation into concrete architectural controls, build a plugin-governance lifecycle that keeps an extensible platform from becoming an ungoverned one, design for the sharpest-edged sector (K-12 edtech, where minors, FERPA, and COPPA collide), and write the governance, accountability, and risk-control specifications a fleet operator and an auditor will actually hold you to.

> **Governance is design, not paperwork.** A control you cannot point at in the architecture is a control you do not have. Every framework clause in this module resolves to a component, a boundary, a log, an approval gate, or an owner — something a reviewer can inspect. If a requirement does not change the system diagram, you have not finished translating it.

## Learning objectives

- **Apply AI governance frameworks** (e.g., ISO/IEC 42001, NIST AI RMF) to an agentic architecture — mapping clauses and functions to concrete components, boundaries, and controls.
- **Translate regulated-domain constraints into architecture**: data privacy, auditability, and accountability become data-flow boundaries, immutable audit logs, and named-owner decision records.
- **Drive plugin/extension governance** for an agentic platform: how third-party and first-party plugins are evaluated, versioned, approved, monitored, and revoked so an extensible platform stays a governed one.
- **Design for sensitive end-users and sectors** — K-12 edtech as the worked case: FERPA and COPPA, safety for minors, human-in-the-loop on student-facing output, and hallucination containment in instructional content.
- **Produce governance, accountability, and risk-control specifications** for agent fleets that an operator can run against and an auditor can review.

## Lecture chapters

1. [Governance Frameworks for Agentic Systems](01-governance-frameworks.md) — ISO/IEC 42001 as an AI management system, the NIST AI RMF functions, and how to map their clauses onto a real agentic architecture.
2. [Regulated-Domain Constraints as Architecture](02-regulated-domain-as-architecture.md) — turning privacy, auditability, and accountability requirements into data-flow boundaries, immutable logs, and decision ownership.
3. [Plugin and Extension Governance](03-plugin-extension-governance.md) — the evaluate → approve → version → monitor → revoke lifecycle that keeps an extensible agent platform from becoming an ungoverned one.
4. [Designing for Sensitive End-Users: K-12 Edtech](04-sensitive-sectors-k12-edtech.md) — FERPA/COPPA, safety for minors, human-in-the-loop on student-facing output, and hallucination containment in instructional content.
5. [Governance, Accountability & Risk-Control Specs for Agent Fleets](05-fleet-governance-and-risk-specs.md) — writing the specifications, RACI, risk register, and control catalog that govern a fleet at scale.

## Exercises

Hands-on practice. These are architecture and specification deliverables, not code. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions).

- [exercise-01: ISO/IEC 42001 controls mapping](exercises/exercise-01-iso-42001-controls-mapping.md) — map the standard's clauses and Annex A controls onto a concrete agentic platform.
- [exercise-02: Plugin lifecycle governance](exercises/exercise-02-plugin-lifecycle-governance.md) — design the evaluate/approve/version/monitor/revoke lifecycle and its gates.
- [exercise-03: Regulated-domain architecture (FERPA/COPPA)](exercises/exercise-03-regulated-domain-architecture-ferpa-coppa.md) — translate the regulations into data-flow boundaries, logging, and accountability.
- [exercise-04: K-12 edtech agentic constraints](exercises/exercise-04-k12-edtech-agentic-constraints.md) — design HITL, minor-safety, and hallucination-containment controls for student-facing agents.
- [exercise-05: Governance & accountability spec](exercises/exercise-05-governance-and-accountability-spec.md) — produce the fleet-level governance, RACI, and risk-control specification.

## Prerequisites

- [mod-306: Guardrails, Safety & Security](../mod-306-guardrails-safety-security/README.md) — the technical controls (input/output filtering, tool scoping, sandboxing) that governance clauses will reference.
- [mod-308: Deployment, Durable Execution & Human-in-the-Loop](../mod-308-deployment-durable-execution/README.md) — HITL approval flows and fleet operation, which this module governs.
- [mod-305: Observability & Tracing](../mod-305-observability-tracing/README.md) — the traces and logs that make auditability real rather than aspirational.

See [resources.md](resources.md) for the primary standards and regulatory sources. **Cite the primary source, not a blog summary** — in governance work, the clause number is the evidence.
