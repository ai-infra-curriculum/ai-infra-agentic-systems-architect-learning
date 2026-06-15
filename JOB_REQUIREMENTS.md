# Agentic Systems Architect — Job Requirements

*Seeded 2026-06-14 from the verbatim Imagine Learning posting that motivated this role. This is one high-signal posting; the nightly research cycle should broaden to ≥25 postings and recompute requirement frequencies. Structured data: [`.aicg/job-requirements.json`](.aicg/job-requirements.json).*

## Reference posting

| Field | Value |
|---|---|
| Employer | Imagine Learning |
| Title | Agentic Systems Architect |
| Posted | 2026-05-19 |
| Location | Remote – US (offices: Tempe AZ HQ, Austin TX, Petaluma CA, Rock Rapids IA, Bloomington MN) |
| Team / reports to | Engineering, Tech & IT / Senior Director of Engineering |
| Salary | $153,415–$200,000/yr + incentive/bonus |
| Experience | 8 yrs (Bachelor's) / 6+ (Master's) / 5+ (PhD) as Lead SWE, Platform Architect, or AI Engineering Lead |
| Work auth | US Person required; no sponsorship |
| Travel | up to ~25% |
| Source | <https://www.edtech.com/jobs/agentic-systems-architect-9085> (original `-8511` expired) |

**Charter (paraphrased from the posting):** Own the design, evolution, and governance of agent-based systems for autonomous/semi-autonomous workflows. The concrete mandate is an **internal AI-for-SDLC platform that extends Claude Code** via a plugin architecture (**Agents, Skills, Hooks, MCP**), letting the AI securely interface with internal SDLC toolchains and proprietary data to perform requirements analysis, code generation, and debugging — **security-first**, **governed**, and **FERPA/COPPA-compliant**.

## Requirement → curriculum traceability

Every requirement the posting lists is mapped to a module. ✅ = covered by a module; 🔗 = owned by a cross-linked role; 🎓 = experience/prerequisite gate (not a teachable module).

| # | Requirement (condensed) | Req/Pref | Covered by | Status |
|---|---|---|---|---|
| 01 | Plugin-based architecture (Agents/Skills/Hooks/MCP) extending Claude Code to interface with SDLC toolchains & proprietary data | Req | `mod-310` | ✅ |
| 02 | Production AI apps via prompt engineering, context management, tool-use | Req | `mod-301`, `mod-303`, `mod-310` | ✅ |
| 03 | Reliable tool-use patterns + context-hydration for autonomous SDLC tasks | Req | `mod-310`, `mod-303` | ✅ |
| 04 | FERPA/COPPA-compliant AI systems in regulated EdTech | Req | `mod-309` | ✅ |
| 05 | Developer-focused tooling: usability, adoption, onboarding, docs, feedback loops | Req | `mod-310` | ✅ |
| 06 | Security-first local/remote agent execution: sandboxing, permission scopes, secure auth for CLI tools | Req | `mod-306` | ✅ |
| 07 | Governance: how plugins are evaluated, versioned, approved, monitored | Req | `mod-309`, `mod-310` | ✅ |
| 08 | Estimation/scoping/planning AI-ML initiatives with execs; Agile/SDLC delivery | Req | `mod-301` + 🔗 `team-lead` track | 🔗 |
| 09 | Continuous improvement of architecture, eng quality, model integration; measurable outcomes | Req | `mod-304`, `mod-305` | ✅ |
| 10 | 8 yrs as Lead SWE / Platform Architect / AI Eng Lead | Req | — (see PREREQUISITES.md) | 🎓 |
| 11 | Built AI dev tooling / function-calling pipelines / extensible agentic platforms in production | Req | `mod-310`, `mod-302` | ✅ |
| 12 | TypeScript/Node.js and/or Python; modular APIs/SDKs/CLI; REST, GraphQL, event-driven | Req | `mod-310` | ✅ |
| 13 | Data privacy/compliance in regulated environments incl. FERPA/COPPA | Req | `mod-309` | ✅ |
| 14 | Secure system design: OAuth, token management, RBAC; mitigating insecure tool execution & prompt injection | Req | `mod-306` | ✅ |
| 15 | Modern SDLC + integrating GitHub, Confluence, Jira, CI/CD | Req | `mod-310` | ✅ |
| 16 | Model Context Protocol (MCP) familiarity ("a major plus") | Pref | `mod-310`, `mod-302` | ✅ |
| 17 | RAG / embedding-based retrieval | Pref | `mod-303` | ✅ |

**Coverage:** 14/15 required *teachable* requirements map to a module; #08 is co-owned with the `team-lead` track (leadership/delivery); #10 is an experience gate recorded in `PREREQUISITES.md`. Both preferred items are covered.

## What this posting added to the generic role

The initial role research described a domain-agnostic "designs multi-agent systems" architect. This specific posting sharpened it toward an **agentic developer-platform / AI-for-SDLC** charter, which drove these curriculum additions:

- **New module `mod-310` — Agentic Developer Platforms & SDLC Integration** (plugin architectures via Agents/Skills/Hooks/MCP, AI-for-SDLC tool-use, context hydration, DevOps toolchain integration, developer DX/adoption).
- **`mod-306` extended** with OAuth / token management / RBAC for toolchains and local+remote CLI-agent sandboxing.
- **`mod-309` extended** with plugin-lifecycle governance (evaluate/version/approve/monitor) alongside ISO/IEC 42001 and FERPA/COPPA.
- **Capstone `project-303`** — design an agentic AI-for-SDLC platform (plugin architecture, secure auth/sandboxing, governance lifecycle, toolchain integrations, adoption plan).
- Track examples provided in **TypeScript/Node.js and Python** (the posting leads with TS/Node).

## Ownership note

Build-altitude fundamentals (agent loops, framework mechanics, RAG/tool-calling implementation) are owned by the lower-level [`agentic-ai-engineer`](../ai-infra-agentic-ai-engineer-learning) track (level 30). This architect track links to them for depth and adds the design/governance/security altitude — it does not re-teach the basics.
