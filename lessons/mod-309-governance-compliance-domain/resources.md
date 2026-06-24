# Resources for mod-309-governance-compliance-domain

Primary references for AI governance and regulated-domain architecture. **Cite the primary source, not a blog summary** — in governance work, the clause number is the evidence. These regimes evolve; verify against the current official text.

The resources lead with the **horizontal** frameworks (which apply to any AI system in any sector), then list the regulated sectors as **balanced peers** — healthcare, finance, public sector, and edtech — none of them the default frame.

## Horizontal AI governance frameworks

- **ISO/IEC 42001 — AI Management System (AIMS)** ([iso.org/standard/42001](https://www.iso.org/standard/42001)) — the certifiable management-system standard: policies, roles, Annex A controls, Plan-Do-Check-Act. The reference model for the clause-to-component mapping in Chapter 1.
- **NIST AI Risk Management Framework (AI RMF 1.0)** ([nist.gov/itl/ai-risk-management-framework](https://www.nist.gov/itl/ai-risk-management-framework)) — the Govern / Map / Measure / Manage functions. The risk *process* of Chapter 1.
- **NIST AI RMF — Generative AI Profile (NIST AI 600-1)** ([nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf)) — maps the RMF onto generative/agentic risks (confabulation, prompt injection, data leakage, autonomy). Directly relevant to the agentic stresses in Chapter 1.
- **EU AI Act — official text (Regulation (EU) 2024/1689)** ([eur-lex.europa.eu/eli/reg/2024/1689/oj](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)) — the risk-tier model (unacceptable / high / limited / minimal) and per-tier obligations. The classification logic reused in Chapter 3.
- **EU AI Act Explorer** ([artificialintelligenceact.eu](https://artificialintelligenceact.eu/)) — navigable companion to the Act's articles and annexes; useful for locating the obligation that attaches to a tier.
- **ISO/IEC 23894 — AI risk management guidance** ([iso.org/standard/77304.html](https://www.iso.org/standard/77304.html)) — complements 42001 with risk-management guidance aligned to ISO 31000; a bridge between 42001 and NIST RMF.
- **OECD AI Principles** ([oecd.ai/en/ai-principles](https://oecd.ai/en/ai-principles)) — the high-level principles (accountability, transparency, robustness) that the frameworks above operationalize.

## Translating constraints into architecture

- **NIST Privacy Framework** ([nist.gov/privacy-framework](https://www.nist.gov/privacy-framework)) — a structured way to turn the *privacy* primitive (Chapter 2) into identify/govern/control/communicate/protect functions.
- **NIST SP 800-53 — Security and Privacy Controls** ([csrc.nist.gov/pubs/sp/800/53/r5/upd1/final](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)) — a deep control catalog for the auditability, accountability, and retention primitives; map clauses to components.
- **Cloud data-residency / sovereignty guidance (provider-neutral)** — consult your cloud provider's data-residency and sovereign-cloud documentation for the *residency* primitive; treat the model endpoint, vector store, tool backends, and trace pipeline all as in-scope (Chapter 2).

## Healthcare (HIPAA) — peer sector

- **HHS — HIPAA Privacy & Security Rules** ([hhs.gov/hipaa](https://www.hhs.gov/hipaa/index.html)) — protected health information (PHI), minimum-necessary, and the safeguards behind the healthcare parameters in Chapters 2–3.
- **HHS — HIPAA Security Rule guidance** ([hhs.gov/hipaa/for-professionals/security](https://www.hhs.gov/hipaa/for-professionals/security/index.html)) — administrative/physical/technical safeguards; maps to the convergent audit-log and access controls.

## Finance (GLBA / SOX / PCI) — peer sector

- **FTC — Gramm-Leach-Bliley Act (GLBA) / Safeguards Rule** ([ftc.gov/business-guidance/privacy-security/gramm-leach-bliley-act](https://www.ftc.gov/business-guidance/privacy-security/gramm-leach-bliley-act)) — financial-data privacy scope.
- **Sarbanes-Oxley (SOX) — SEC overview** ([sec.gov](https://www.sec.gov/)) — internal-control and audit-record integrity/retention; the SOX "attestation" structural delta in Chapter 3.
- **PCI DSS — PCI Security Standards Council** ([pcisecuritystandards.org](https://www.pcisecuritystandards.org/)) — cardholder-data handling and scope reduction; the PCI "card-data scope" structural delta in Chapter 3.

## Public sector — peer sector

- **NIST SP 800-53 / FedRAMP** ([fedramp.gov](https://www.fedramp.gov/)) — public-sector control baselines and authorization; the residency/sovereignty and auditability emphasis in Chapter 3.
- **Public-records and transparency law (jurisdiction-specific)** — consult the applicable records-management, freedom-of-information, and procurement rules for your jurisdiction; these drive the "public-records disclosure" structural delta.

## Edtech (FERPA / COPPA) — peer sector

- **U.S. Dept. of Education — FERPA** ([studentprivacy.ed.gov](https://studentprivacy.ed.gov/)) — education-record privacy and the institutional-accountability parameters.
- **FTC — COPPA (Children's Online Privacy Protection)** ([ftc.gov/legal-library/browse/rules/childrens-online-privacy-protection-rule-coppa](https://www.ftc.gov/legal-library/browse/rules/childrens-online-privacy-protection-rule-coppa)) — verifiable parental consent for minors; the "minor-consent" structural delta in Chapter 3.

## Cross-border privacy (a fifth peer to test convergence)

- **GDPR — official text** ([gdpr-info.eu](https://gdpr-info.eu/)) — data-subject rights (including erasure), purpose limitation, and cross-border transfer; a useful fifth regime for the stretch goals in exercises 02 and 05.

## Extension and fleet governance

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — where human checkpoints and tool boundaries sit in the agent taxonomy; framing for the gate-placement and extension-governance discipline in Chapters 4–5.
- **Model Context Protocol (MCP)** ([modelcontextprotocol.io](https://modelcontextprotocol.io/)) — the interface model for third-party tools/servers the extension lifecycle (Chapter 4) must govern: version-pin, scope, monitor, revoke.
- **OWASP Top 10 for LLM Applications** ([owasp.org/www-project-top-10-for-large-language-model-applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)) — the supply-chain, excessive-agency, and data-leakage risks that the Evaluate gate and monitoring of Chapter 4 are designed to catch.

## Within this track

- [mod-305: Observability & Tracing](../mod-305-observability-tracing/README.md) — the traces and metrics that become audit evidence and drift signals.
- [mod-306: Guardrails, Safety & Security](../mod-306-guardrails-safety-security/README.md) — the technical controls a framework clause points at.
- [mod-308: Deployment, Durable Execution & Human-in-the-Loop](../mod-308-deployment-durable-execution/README.md) — the durable HITL gates and immutable run history that lineage, audit trails, and accountability points build on.
