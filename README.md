# AI Infrastructure Agentic Systems Architect — Learning Repository

<!-- aicg:site-banner -->
> 🎓 Part of the **free, open-source AI Infrastructure Curriculum**. For live, instructor-led **[cohorts](https://ai-infra-curriculum.github.io/junior.html)** and **[team programs](https://ai-infra-curriculum.github.io/teams.html)**, visit **[ai-infra-curriculum.github.io](https://ai-infra-curriculum.github.io/)**.
<!-- /aicg:site-banner -->

A design-altitude (L48) learning path for the **Agentic Systems Architect** — the engineer who owns the end-to-end architecture of production agentic AI systems rather than building any single agent loop.

> **Status**: ✅ Curriculum complete — 10 modules with lecture chapters, exercises, and quizzes authored. AI-assisted content under ongoing human review.

---

## 🎯 Overview

This track is for engineers who already know how to *build* agents and now need to *architect* them at production scale. It deliberately sits one altitude above the build-level fundamentals (agent loops, framework mechanics, RAG implementation) owned by the lower [Senior Agentic AI Engineer](#-prerequisites) rung — those are linked, not re-taught.

The 10 modules form the **production-hardening spine** that introductory agent courses skip:

- **Orchestration topology** — workflows vs. agents, orchestrator-worker patterns, handoffs vs. manager-as-tools, MCP and A2A protocol boundaries, escalation and coordination design.
- **Memory & context architecture** — context engineering, context-rot mitigation, working/episodic/long-term memory placement, RAG and knowledge-graph stores as an architectural concern.
- **Evaluation harnesses** — trajectory and tool-call-correctness evaluation, LLM-as-judge rubrics, and eval-gated release pipelines.
- **Observability & tracing** — OpenTelemetry GenAI spans, stable trace identity across retries and long-running sessions, quality and drift signals.
- **Guardrails, safety & security** — guardrail placement, prompt-injection threat modeling, least-privilege tool permissions, OAuth/RBAC token management, CLI-agent sandboxing, and excessive-agency controls.
- **Cost & latency architecture** — token economics, caching and routing, and cost/latency/quality budgets.
- **Deployment & durable execution** — durable workflows, human-in-the-loop approval boundaries, agent-fleet deployment, failure recovery and resumption.
- **Governance & compliance** — ISO/IEC 42001 control mapping, plugin lifecycle governance, regulated-domain (FERPA/COPPA, K-12 EdTech) constraints, and accountability specs.
- **Agentic developer platforms / AI-for-SDLC** — plugin architecture (agents, skills, hooks, MCP), tool-use patterns, context hydration, DevOps-toolchain integration, and developer-experience adoption.

Throughout, you decide *whether and how* each capability belongs in the architecture — the discipline of "start simple, add complexity only when it demonstrably improves outcomes."

---

## 📚 Modules

| Module | Topic | Hours | Exercises | Quiz |
|--------|-------|-------|-----------|------|
| [mod-301](./lessons/mod-301-agentic-systems-foundations/README.md) | Agentic Systems Foundations & When NOT to Use an Agent | 12 | 4 | Yes |
| [mod-302](./lessons/mod-302-multi-agent-orchestration/README.md) | Multi-Agent Orchestration Architecture | 14 | 4 | Yes |
| [mod-303](./lessons/mod-303-memory-context-architecture/README.md) | Memory & Context Architecture | 12 | 4 | Yes |
| [mod-304](./lessons/mod-304-evaluation-harnesses/README.md) | Evaluation Harnesses for Agentic Systems | 12 | 4 | Yes |
| [mod-305](./lessons/mod-305-observability-tracing/README.md) | Observability & Tracing for Agents | 10 | 3 | Yes |
| [mod-306](./lessons/mod-306-guardrails-safety-security/README.md) | Guardrails, Safety & Security | 20 | 6 | Yes |
| [mod-307](./lessons/mod-307-cost-latency-architecture/README.md) | Cost & Latency Architecture | 10 | 3 | Yes |
| [mod-308](./lessons/mod-308-deployment-durable-execution/README.md) | Deployment, Durable Execution & Human-in-the-Loop | 12 | 4 | Yes |
| [mod-309](./lessons/mod-309-governance-compliance-domain/README.md) | Governance, Compliance & Domain Constraints | 15 | 5 | Yes |
| [mod-310](./lessons/mod-310-agentic-developer-platforms/README.md) | Agentic Developer Platforms & SDLC Integration | 16 | 5 | Yes |

**Totals:** 10 modules · 133 hours · 42 exercises · 10 quizzes.

---

## 🛠️ Projects

Multi-module capstones that integrate the spine into end-to-end architecture deliverables.

| Project | Focus | Hours |
|---------|-------|-------|
| [project-301](./projects/project-301-production-agentic-reference-architecture/README.md) | End-to-End Agentic Reference Architecture (capstone) | 30 |
| [project-302](./projects/project-302-regulated-domain-agent-architecture/README.md) | Regulated-Domain Agentic Architecture (e.g., K-12 EdTech) | 20 |
| [project-303](./projects/project-303-agentic-sdlc-platform/README.md) | Agentic AI-for-SDLC Platform (Plugin Architecture) | 25 |

---

## 🎓 Prerequisites

This is the **architect** rung. It assumes build-level mastery of agentic systems — agent loops, framework mechanics, tool use, and RAG implementation — which are owned by the lower rung:

- **[Senior Agentic AI Engineer](https://github.com/ai-infra-curriculum/ai-infra-senior-agentic-ai-engineer-learning)** (L40) — the recommended on-ramp.

You should be able to *build* a multi-tool agent and a working RAG pipeline before starting; here you decide *whether, where, and how* those pieces belong in a production architecture. See [PREREQUISITES.md](./PREREQUISITES.md) for the full role-level entry-skill checklist.

---

## 🚀 Getting Started

```bash
# 1. Clone the repository
git clone https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-learning.git
cd ai-infra-agentic-systems-architect-learning

# 2. Confirm you meet the entry skills
cat PREREQUISITES.md

# 3. Start with Module 301
cd lessons/mod-301-agentic-systems-foundations
cat README.md
```

Work the modules in order — each builds on the orchestration and memory foundations laid in mod-301 through mod-303. Each module README lists its objectives, lecture chapters, exercises, and a knowledge-check quiz. Tackle the projects after the modules they integrate.

Examples appear in both **TypeScript/Node.js** and **Python** — agentic developer tooling skews heavily toward the TS/Node ecosystem, while data and evaluation tooling skews Python.

---

## 📖 Curriculum Overview

### mod-301: Agentic Systems Foundations & When NOT to Use an Agent

Distinguish workflows from agents and justify the choice per task; apply the orchestration patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer); decompose a system into tool vs. sub-agent vs. hardcoded-path boundaries.

### mod-302: Multi-Agent Orchestration Architecture

Design orchestrator-worker topologies and reason about their token economics (~15x cost vs. a single chat); choose between handoffs and manager-as-tools; apply MCP and A2A protocols at the architecture level; define escalation and coordination strategies.

### mod-303: Memory & Context Architecture

Architect context-engineering strategies (compaction, structured note-taking, sub-agent isolation, just-in-time retrieval); diagnose and mitigate context rot in long-horizon agents; decide where state lives across working/episodic/long-term memory; integrate RAG and vector/knowledge-graph stores.

### mod-304: Evaluation Harnesses for Agentic Systems

Design trajectory and tool-call-correctness evaluations; build calibrated LLM-as-judge rubrics; gate releases behind an eval pipeline that compares trajectory quality, not just final-state outputs.

### mod-305: Observability & Tracing for Agents

Architect tracing for non-deterministic, long-running agents — OpenTelemetry GenAI spans, stable trace identity across retries, sessions spanning hours, and comparability across model and prompt deploys; instrument quality and drift signals.

### mod-306: Guardrails, Safety & Security

Place guardrails across the request path; threat-model prompt injection; enforce least-privilege tool permissions, OAuth/RBAC token management for toolchains, and CLI-agent sandboxing (local and remote); design excessive-agency controls.

### mod-307: Cost & Latency Architecture

Build token-economics cost models; architect caching and routing; define and defend cost/latency/quality budgets across an agentic system.

### mod-308: Deployment, Durable Execution & Human-in-the-Loop

Design durable execution for long-running agents; architect human-in-the-loop approval boundaries; plan agent-fleet deployment strategies and failure recovery and resumption.

### mod-309: Governance, Compliance & Domain Constraints

Map ISO/IEC 42001 controls; govern the plugin lifecycle; architect regulated-domain systems under FERPA/COPPA and K-12 EdTech constraints; produce fleet-level governance, RACI, and risk-control specifications.

### mod-310: Agentic Developer Platforms & SDLC Integration

Design plugin architectures (agents, skills, hooks, MCP); apply AI-for-SDLC tool-use patterns; architect context-hydration strategies; integrate the DevOps toolchain and drive developer-tooling adoption and DX.

---

## 🔗 Paired Solutions Repo

Reference implementations and worked architecture deliverables live in the paired solutions repository:

- [`ai-infra-agentic-systems-architect-solutions`](https://github.com/ai-infra-curriculum/ai-infra-agentic-systems-architect-solutions)

---

<!-- aicg:maintained-by -->
Maintained by [VeriSwarm.ai](https://veriswarm.ai)
