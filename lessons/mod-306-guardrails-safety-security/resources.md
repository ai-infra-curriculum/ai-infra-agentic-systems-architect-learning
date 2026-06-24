# Resources for mod-306-guardrails-safety-security (Guardrails, Safety & Security)

Primary references for agentic safety and security architecture. Security guidance evolves quickly — verify against current versions, especially the OWASP list and provider docs.

## OWASP — the anchor texts

- **OWASP Top 10 for LLM Applications 2025** ([genai.owasp.org/llm-top-10](https://genai.owasp.org/llm-top-10/)) — the canonical risk taxonomy for this module. Read the full list; this module centers on two entries.
- **LLM01:2025 Prompt Injection** ([genai.owasp.org/llmrisk/llm01-prompt-injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)) — direct and indirect injection, why prevention is impossible, and the recommended mitigations. The reference for Chapters 2 and 6.
- **LLM06:2025 Excessive Agency** ([genai.owasp.org/llmrisk/llm062025-excessive-agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)) — excessive functionality, permissions, and autonomy, and the controls for each. The reference for exercise-06.
- **OWASP — Agentic AI: Threats and Mitigations** ([genai.owasp.org/resource/agentic-ai-threats-and-mitigations](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/)) — agent-specific threat catalog beyond the base Top 10.

## Anthropic — safety and prompt injection

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the patterns you are securing; the guardrail and human-in-the-loop guidance is directly relevant.
- **Anthropic — Mitigating prompt injection and other risks** ([docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)) — practical layered defenses (input/output screening, tool restrictions).
- **Anthropic — Reducing the risk of prompt injection in computer use** ([docs.anthropic.com/en/docs/agents-and-tools/computer-use](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use)) — containment guidance where the agent acts on untrusted screen/web content.
- **Anthropic — Responsible Scaling Policy** ([anthropic.com/rsp](https://www.anthropic.com/rsp)) — the safety-by-design framing for high-capability systems.

## OAuth, tokens, and RBAC

- **OAuth 2.1 (draft, consolidating 2.0 best practice)** ([oauth.net/2.1](https://oauth.net/2.1/)) — the modern baseline: Authorization Code + PKCE, no implicit grant.
- **RFC 6749 — The OAuth 2.0 Authorization Framework** ([datatracker.ietf.org/doc/html/rfc6749](https://datatracker.ietf.org/doc/html/rfc6749)) — the normative spec for the flows in Chapter 4.
- **RFC 7636 — PKCE** ([datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636)) — Proof Key for Code Exchange.
- **RFC 8693 — OAuth 2.0 Token Exchange** ([datatracker.ietf.org/doc/html/rfc8693](https://datatracker.ietf.org/doc/html/rfc8693)) — downscoping a token per tool.
- **RFC 9700 — Best Current Practice for OAuth 2.0 Security** ([datatracker.ietf.org/doc/html/rfc9700](https://datatracker.ietf.org/doc/html/rfc9700)) — token handling, audience binding, and current threat guidance.
- **NIST SP 800-162 — Guide to Attribute Based Access Control** ([csrc.nist.gov/pubs/sp/800/162/upd2/final](https://csrc.nist.gov/pubs/sp/800/162/upd2/final)) — RBAC/ABAC reference for the authorization layer.
- **Model Context Protocol — Authorization** ([modelcontextprotocol.io/specification/draft/basic/authorization](https://modelcontextprotocol.io/specification/draft/basic/authorization)) — OAuth-based auth for MCP tool servers, the agent-toolchain case directly.

## Sandboxing and isolation

- **gVisor** ([gvisor.dev](https://gvisor.dev/)) — userspace kernel for sandboxing untrusted code; relevant to remote CLI-agent isolation.
- **Firecracker microVMs** ([firecracker-microvm.github.io](https://firecracker-microvm.github.io/)) — lightweight VM isolation for stronger-than-container boundaries.
- **seccomp-bpf** ([docs.kernel.org/userspace-api/seccomp_filter.html](https://docs.kernel.org/userspace-api/seccomp_filter.html)) — syscall filtering for the process-hardening dimension.
- **OWASP — Container Security Cheat Sheet** ([cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)) — read-only fs, dropped caps, no-new-privileges, non-root.

## Threat modeling and red-teaming

- **Microsoft — STRIDE threat modeling** ([learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)) — the STRIDE categories used in exercise-02.
- **NIST AI 600-1 — Generative AI Profile (AI RMF)** ([nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf)) — risk-management framing, including red-teaming for GenAI.
- **Simon Willison — Prompt injection and the dual-LLM pattern** ([simonwillison.net/2023/Apr/25/dual-llm-pattern](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/)) — the quarantine/privileged split in Chapter 6, explained at the source.
- **MITRE ATLAS** ([atlas.mitre.org](https://atlas.mitre.org/)) — adversarial tactics and techniques for AI systems, useful for building the adversarial test suite.

> You designed these controls as placed, deterministic components. When you adopt a framework's built-in guardrails or a managed policy engine, you will recognize exactly which boundary it covers — and, more importantly, which it does not. A safety property you cannot draw and test is one you cannot defend.
