# Agentic Systems Architect — Job Requirements

*Generated 2026-06-15 (Cycle 2). Last cycle (2026-06-14) seeded the role from a single high-signal Imagine Learning posting. This cycle broadened the sample to 22 in-window postings (≥ 2026-03-17) plus 5 corroborating older postings. Structured data: [`.aicg/job-requirements.json`](.aicg/job-requirements.json). Cycle-over-cycle delta: [`.aicg/curriculum-plan-delta.json`](.aicg/curriculum-plan-delta.json).*

## Cycle-2 verdict

**No curriculum changes proposed.** Every requirement that crosses the 0.30 frequency bar maps to an existing module. The one borderline gap (TH-08, "MCP server / tool registry / agent gateway as a governance pattern", 6/22 = 27%) sits just below threshold and is already addressed by the combination of `mod-306` + `mod-309` + `mod-310`. Per the continuity-bias rule, the expected output is empty deltas.

What did move:
- `REQ-ASA-16` (Model Context Protocol) is **promoted** from preferred to required-tier evidence (9/22 in-window postings cite MCP directly). No curriculum change is needed — `mod-302`, `mod-306`, and `mod-310` already cover MCP at architect altitude.
- Two requirement entries added as `required_emerging` (`REQ-ASA-18` agent observability, `REQ-ASA-19` cross-cloud agent protocols / managed agent platforms) to reflect the consolidation seen in this cycle's evidence. Both already covered by existing modules.

## Postings sampled (27 total; 22 in 90-day window)

| # | Employer | Title | Date | Location | Window? |
|---|---|---|---|---|---|
| 01 | Imagine Learning | Agentic Systems Architect | 2026-05-19 | Remote US | ✓ |
| 02 | NTT DATA Services | AI Architect Director - Agentic Systems | 2026-05-21 | Dallas TX | ✓ |
| 03 | Cognizant | Solution Architect - Agentic AI | 2026-06-11 | Stockholm SE | ✓ |
| 04 | Cognizant | Principal AI Architect — Agentic AI, AWS Bedrock & Broad-Spectrum AI (Remote) | 2026-06-11 | Dallas TX | ✓ |
| 05 | Guidehouse | AI/ML Solutions Architect (Agentic Systems) | 2026-04-09 | Remote US | ✓ |
| 06 | Infosys | Agentic AI Architect | 2026-06-07 | Remote Poland | ✓ |
| 07 | HERE Technologies | Agentic AI Domain Architect | 2026-06-04 | Hybrid | ✓ |
| 08 | Zoom | Principal AI Architect - Agentic Verticals | 2026-06-12 | Seattle / San Jose | ✓ |
| 09 | Granicus | Agentic AI Architect & Value Engineer | 2026-03-15 | Remote US | ✓ |
| 10 | WGA Consulting | Principal AI Agent Architect | 2026-05-25 | El Segundo CA | ✓ |
| 11 | Experian | Director, Agentic AI/Automation | 2026-05-20 | Remote US | ✓ |
| 12 | Zscaler | Security Architect, Agentic AI | 2026-05-15 | Remote US | ✓ |
| 13 | Acquia | Enterprise AI Architect | 2026-05-10 | Remote Canada | ✓ |
| 14 | Scale AI | Principal Architect | 2026-04-25 | Washington DC | ✓ |
| 15 | ABCO Computers (engagement: 84.51° / Kroger AI Enablement) | AI Architect (Agentic Platforms) | 2026-05-30 | Blue Ash OH | ✓ |
| 16 | SIA Innovations | Agentic AI Engineer/Architect — FTA | 2026-05-05 | Montréal QC | ✓ |
| 17 | Anthropic | Manager of Applied AI Architecture, Partnerships | 2026-05-28 | SF / NYC / Seattle | ✓ |
| 18 | Anthropic | Manager of Applied AI Architecture, Enterprise Tech (Cyber) | 2026-05-30 | NYC / SF / Seattle | ✓ |
| 19 | UnitedHealth Group | Director of Architecture - Agentic AI Claims Automation | 2026-05-08 | Eden Prairie MN | ✓ |
| 20 | 84.51° (Kroger) | AI Architect - Agentic Platforms | 2026-05-12 | Cincinnati / Chicago / NYC | ✓ |
| 21 | Cognizant | Agentic AI Architect (Richardson, TX) | 2026-04-20 | Richardson TX | ✓ |
| 22 | Cognizant | Senior Manager - Agentic AI Architect | 2026-05-02 | Chennai IN | ✓ |
| 23 | Capgemini | Agentic Architect | 2026-04-15 | Atlanta GA | ✓ |
| 24 | 66degrees | Workspace Architect (incl. AI Agent solutions on GCP) | 2026-05-18 | Remote US | ✓ |
| 25 | dentsu (Merkle) | Agentic AI Architect | 2025-11-13 | Remote OH | ✗ corroborating |
| 26 | Vertex, Inc. | Principal Architect, Agentic AI | 2026-01-09 | Remote US | ✗ corroborating |
| 27 | NVIDIA | Solutions Architect, Agentic AI | 2025-05-30 | 4 locations | ✗ corroborating |

The `.aicg/job-requirements.json` file holds employer, URL, verbatim quote snippets, salary where disclosed, and a `source_depth` flag (`full_jd` vs `search_result_only`) for each row.

## Theme frequency (computed on 22 in-window postings)

| Theme | Postings | Freq | Covered by | Notes |
|---|---:|---:|---|---|
| TH-01 Multi-agent orchestration architecture | 18 | 0.82 | `mod-302`, `mod-301` | — |
| TH-02 Framework fluency (LangGraph/CrewAI/AutoGen/SK/LC/LI) | 16 | 0.73 | `mod-302`, `mod-301` | Implementation-altitude owned by `agentic-ai-engineer` (level 30) |
| TH-04 Memory / context / RAG | 15 | 0.68 | `mod-303` | — |
| TH-09 AI governance / RAI / ISO-42001 / plugin lifecycle | 12 | 0.55 | `mod-309` | — |
| TH-07 Secure agent execution (OAuth/RBAC, sandboxing, OWASP LLM Top 10, MITRE ATLAS, NIST AI RMF) | 11 | 0.50 | `mod-306` | — |
| TH-13 Pre-sales / executive scoping / consulting | 10 | 0.45 | `mod-301` + 🔗 `team-lead` | — |
| TH-05 Evaluation harnesses (trajectory, tool-call, LLM-as-judge, regression gates) | 10 | 0.45 | `mod-304` | — |
| TH-03 MCP integration | 9 | 0.41 | `mod-310`, `mod-302` | **Promoted** from preferred → required-tier this cycle |
| TH-06 Observability / tracing (OTel GenAI, LangFuse, LangSmith, Arize) | 9 | 0.41 | `mod-305` | — |
| TH-15 Bedrock AgentCore / Azure OpenAI / Vertex AI Agent Builder as platform-of-record | 8 | 0.36 | `mod-302`, `mod-310` | Platform-agnostic by design |
| TH-10 Regulated-domain (FERPA/COPPA, HIPAA, FedRAMP, federal Public Trust) | 8 | 0.36 | `mod-309` | Seed FERPA/COPPA exercise still primary; broader sample adds HIPAA/federal context — absorbed by `project-302` capstone |
| TH-08 MCP server / tool registry / agent gateway (enterprise tool catalog) | 6 | 0.27 | `mod-306`, `mod-309`, `mod-310` | **Below threshold.** Composite of three existing modules. Monitor next cycle. |
| TH-12 Durable execution / HITL / long-running deployment | 6 | 0.27 | `mod-308` | — |
| TH-11 Cost/latency/quality budgets | 5 | 0.23 | `mod-307` | — |
| TH-14 Plugin architectures extending agentic coding tools (AI-for-SDLC) | 3 | 0.14 | `mod-310` | One named domain, not the whole role |

## Requirement → curriculum traceability (cycle 2)

✅ covered by module; 🔗 cross-linked to another role; 🎓 experience/prerequisite gate.

| # | Requirement (condensed) | Type | Covered by | Status |
|---|---|---|---|---|
| 01 | Plugin architecture (Agents/Skills/Hooks/MCP) extending an agentic coding tool to interface with SDLC toolchains & proprietary data | Req | `mod-310` | ✅ |
| 02 | Production AI apps via prompt engineering, context management, tool-use | Req | `mod-301`, `mod-303`, `mod-310` | ✅ |
| 03 | Reliable tool-use + context-hydration for autonomous tasks | Req | `mod-310`, `mod-303` | ✅ |
| 04 | Data privacy / regulatory compliance (FERPA, COPPA, HIPAA, FedRAMP, federal Public Trust) | Req | `mod-309` | ✅ |
| 05 | Developer-focused tooling: usability, adoption, onboarding, docs, feedback loops | Req | `mod-310` | ✅ |
| 06 | Security-first agent execution: sandboxing, permission scopes, secure auth; OWASP LLM Top 10 / MITRE ATLAS / NIST AI RMF | Req | `mod-306` | ✅ |
| 07 | Governance: how plugins/agents/tools are evaluated, versioned, approved, monitored; align with ISO/IEC 42001 & RAI | Req | `mod-309`, `mod-310` | ✅ |
| 08 | Estimation/scoping/planning with execs; Agile/SDLC; client/pre-sales scoping | Req | `mod-301` + 🔗 `team-lead` | 🔗 |
| 09 | Continuous improvement of architecture & model integration via measurable outcomes (eval + observability) | Req | `mod-304`, `mod-305` | ✅ |
| 10 | 7–15+ yrs as Lead SWE / Platform Architect / AI Eng Lead | Req | — (see [`PREREQUISITES.md`](PREREQUISITES.md)) | 🎓 |
| 11 | Production AI dev tooling / function-calling pipelines / extensible agentic platforms; modular, scalable architecture | Req | `mod-310`, `mod-302` | ✅ |
| 12 | Python and TypeScript/Node.js; modular APIs/SDKs/CLI; REST, GraphQL, event-driven | Req | `mod-310` | ✅ |
| 13 | Data privacy/compliance in regulated environments | Req | `mod-309` | ✅ |
| 14 | Secure system design: OAuth, token management, RBAC; mitigating insecure tool execution & prompt injection | Req | `mod-306` | ✅ |
| 15 | Modern SDLC + integrating GitHub, Confluence, Jira, CI/CD | Req | `mod-310` | ✅ |
| 16 | **Model Context Protocol (MCP)** — build/secure MCP servers, tool registries, connector patterns | **Req (↑promoted)** | `mod-310`, `mod-302`, `mod-306` | ✅ |
| 17 | RAG / embedding-based retrieval | Pref | `mod-303` | ✅ |
| 18 | Architect agent observability: tracing/eval/prompt management; OTel GenAI; LangFuse/LangSmith/Arize | Req (emerging) | `mod-305`, `mod-304` | ✅ |
| 19 | Cross-cloud agent protocols & managed agent platforms (A2A; Bedrock AgentCore; Azure AI Foundry; Vertex AI Agent Builder) | Req (emerging) | `mod-302`, `mod-310` | ✅ |

**Coverage:** 17/17 required *teachable* items map to a module. #08 remains co-owned with `team-lead`. #10 stays in `PREREQUISITES.md`. The preferred item (#17) is covered. Two new emerging requirements (#18, #19) are absorbed by the existing modules without curriculum changes.

## What changed cycle-over-cycle

- **Sample size:** 1 → 27 (22 in 90-day window).
- **MCP promoted:** was preferred ("a major plus"); now appears in 9/22 in-window postings as a required-tier capability. Curriculum already covered it at architect altitude.
- **Two new emerging requirements named** (`REQ-ASA-18` observability platform architecture; `REQ-ASA-19` cross-cloud agent protocols / managed agent platforms). Both already covered.
- **Regulated-domain coverage broadened** in the evidence (FERPA/COPPA seed + HIPAA + FedRAMP / federal Public Trust). `mod-309` and `project-302` remain framework-agnostic and absorb this without restructure.
- **Borderline gap watched:** TH-08 (agent gateway / tool registry / MCP server catalog as enterprise governance pattern) is at 27% — under the 0.30 bar. If it crosses next cycle, the cleanest path is a single new exercise inside `mod-309` (`agent-gateway-and-tool-registry-governance`) rather than a new module.

## What this role is NOT (avoiding scope creep)

Postings in this cycle cluster into three sub-flavors — consulting/SI, enterprise-platform, and security/governance — but they share the same architect-altitude skill set. Implementation-altitude framework mechanics (LangGraph DSL details, vector-store internals, RAG implementation) are owned by the lower-level [`agentic-ai-engineer`](../ai-infra-agentic-ai-engineer-learning) track (level 30). Leadership/delivery depth (estimation, cross-team execution, people management) is co-owned with the [`team-lead`](../ai-infra-team-lead-learning) track (level 40). This architect track stays at the design/governance/security altitude and links down rather than re-teaching.

## Unresolved items

<!-- needs-research: Confirm exact post date for Zoom "Principal AI Architect - Agentic Verticals" — listed as "Reposted 3 Days Ago" relative to 2026-06-15; treated as 2026-06-12. -->
<!-- needs-research: 84.51° / Kroger "AI Architect - Agentic Platforms" full JD; index page returned aggregate listing. Cross-checked against ABCO/Blue Ash contractor JD whose text appears to mirror the same engagement. -->
<!-- needs-research: UnitedHealth Group "Director of Architecture - Agentic AI Claims Automation" direct fetch returned 404; req captured via cached aggregator summary only. -->
<!-- needs-research: Capgemini Atlanta "Agentic Architect" and Cognizant Richardson "Agentic AI Architect" pages 404'd (likely filled); included on cached search summaries. -->
