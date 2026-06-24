# Resources for mod-309-governance-compliance-domain

Primary sources for AI governance, compliance, and domain constraints. **In governance work, cite the primary source — the clause or section number is the evidence.** Verify against the current text; standards and regulations are revised.

## AI governance frameworks

- **ISO/IEC 42001:2023 — Information technology — Artificial intelligence — Management system** ([iso.org/standard/42001](https://www.iso.org/standard/42001.html)) — the certifiable AI management system (AIMS) standard. Clauses 4–10 are the auditable requirements; Annex A is the reference control set; Annex B is implementation guidance; Annex C lists objectives and risk sources. The normative text is paywalled; purchase or access via your organization's standards subscription.
- **NIST AI Risk Management Framework (AI RMF 1.0), NIST AI 100-1** ([nist.gov/itl/airc/ai-risk-management-framework](https://www.nist.gov/itl/airc/ai-risk-management-framework) · PDF: [doi.org/10.6028/NIST.AI.100-1](https://doi.org/10.6028/NIST.AI.100-1)) — the GOVERN / MAP / MEASURE / MANAGE functions. Free. Start here for operational risk vocabulary.
- **NIST AI RMF Generative AI Profile, NIST AI 600-1** ([doi.org/10.6028/NIST.AI.600-1](https://doi.org/10.6028/NIST.AI.600-1)) — risks specific to generative (and agentic) systems: confabulation/hallucination, data leakage, harmful content, automation-amplified harm. Use as the risk catalog for the MAP function.
- **NIST AI RMF Playbook & crosswalks** ([airc.nist.gov/AI_RMF_Knowledge_Base/Playbook](https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook)) — actionable suggestions per subcategory and crosswalks to ISO/IEC 42001 and other frameworks.

## Privacy regulations (primary sources — U.S.)

- **FERPA — Family Educational Rights and Privacy Act**, 20 U.S.C. § 1232g ([law.cornell.edu/uscode/text/20/1232g](https://www.law.cornell.edu/uscode/text/20/1232g)); implementing regulations at **34 CFR Part 99** ([ecfr.gov/current/title-34/subtitle-A/part-99](https://www.ecfr.gov/current/title-34/subtitle-A/part-99)). The "school official" exception is **34 CFR § 99.31(a)(1)**. U.S. Dept. of Education Student Privacy Policy Office: [studentprivacy.ed.gov](https://studentprivacy.ed.gov).
- **COPPA — Children's Online Privacy Protection Act**, 15 U.S.C. §§ 6501–6506 ([law.cornell.edu/uscode/text/15/chapter-91](https://www.law.cornell.edu/uscode/text/15/chapter-91)); the FTC's **COPPA Rule at 16 CFR Part 312** ([ecfr.gov/current/title-16/chapter-I/subchapter-C/part-312](https://www.ecfr.gov/current/title-16/chapter-I/subchapter-C/part-312)). FTC guidance hub: [ftc.gov/business-guidance/privacy-security/childrens-privacy](https://www.ftc.gov/business-guidance/privacy-security/childrens-privacy). See the FTC's "COPPA and Schools" guidance for school-provided consent in the educational context.
- **HIPAA — Health Insurance Portability and Accountability Act**, Privacy Rule at **45 CFR Parts 160 and 164** ([hhs.gov/hipaa](https://www.hhs.gov/hipaa)) — for the pediatric-health analogue of the K-12 case.
- **CCPA/CPRA — California Consumer Privacy Act, as amended** ([oag.ca.gov/privacy/ccpa](https://oag.ca.gov/privacy/ccpa)); **SOPIPA — Student Online Personal Information Protection Act**, Cal. B&P Code §§ 22584–22585 — a state-law layer on top of FERPA/COPPA for California districts.

## Privacy regulations (primary sources — EU)

- **GDPR — General Data Protection Regulation (EU) 2016/679** ([eur-lex.europa.eu/eli/reg/2016/679/oj](https://eur-lex.europa.eu/eli/reg/2016/679/oj)) — Article 22 restricts solely-automated decisions with significant effects; principles in Articles 5–6 (minimization, purpose limitation, lawful basis) map directly to Chapter 2's boundaries.
- **EU AI Act — Regulation (EU) 2024/1689** ([eur-lex.europa.eu/eli/reg/2024/1689/oj](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)) — the risk-tiered AI regulation: prohibited practices, high-risk system obligations (risk management, data governance, human oversight, logging, transparency), and GPAI provisions. The risk-tiering and human-oversight requirements parallel this module's tier model and HITL gates.

## Mapping standards to architecture

- **NIST AI RMF Knowledge Base** ([airc.nist.gov](https://airc.nist.gov)) — crosswalks, use-case profiles, and the trustworthiness characteristics (valid/reliable, safe, secure, accountable, explainable, privacy-enhanced, fair) that translate into controls.
- **ISO/IEC 23894:2023 — Guidance on AI risk management** ([iso.org/standard/77304](https://www.iso.org/standard/77304.html)) — companion risk-management guidance to ISO/IEC 42001.
- **ISO/IEC 27001 / 27002** ([iso.org/standard/27001](https://www.iso.org/standard/27001.html)) — the ISMS the AIMS bolts onto; its Annex A structure is the model for ISO/IEC 42001's.

## Agent-specific governance and safety

- **OWASP Top 10 for LLM Applications & GenAI** ([genai.owasp.org](https://genai.owasp.org)) — concrete threat catalog (prompt injection, insecure plugin design, excessive agency, supply-chain) that your plugin-governance and tool-scoping controls mitigate.
- **NIST SP 800-218A — Secure Software Development Practices for Generative AI** ([doi.org/10.6028/NIST.SP.800-218A](https://doi.org/10.6028/NIST.SP.800-218A)) — life-cycle secure-development practices that feed the 42001 Annex A life-cycle controls.
- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the agent patterns this module governs; read for the architecture being constrained.

> Many of these (ISO standards in particular) are paywalled. Where you cannot obtain the normative text, use NIST AI 100-1 and the official regulatory texts on eCFR/EUR-Lex — they are free, authoritative, and citable by section number.
