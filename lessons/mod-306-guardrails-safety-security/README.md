# mod-306-guardrails-safety-security: Guardrails, Safety & Security

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 20 hours

## Learning objectives

- Design two-sided input/output moderation and per-tool-call guardrails
- Mitigate OWASP LLM01:2025 (prompt injection, incl. indirect) and LLM06:2025 (excessive agency)
- Apply least-privilege tool access, tool sandboxing, and human approval for high-risk actions
- Architect secure authentication and authorization for agent toolchains: OAuth flows, token management, and Role-Based Access Control (RBAC)
- Design sandboxing and strict permission scopes for local and remote agent/CLI execution to prevent destructive actions or data leakage
- Architect segregation of untrusted content and adversarial testing into the system

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
