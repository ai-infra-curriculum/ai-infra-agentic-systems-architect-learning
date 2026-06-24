# exercise-04: OAuth, RBAC, and Token Management for Toolchains

**Estimated effort:** 3 hours

## Objective

Design the **delegated-authorization and token-lifecycle architecture** for a multi-tool agent that reaches real external systems. You will choose the right OAuth flow per integration, design the token lifecycle so a leaked token is near-useless, and layer RBAC over the tool catalog so the agent's effective authority is the intersection of three independent narrowings. By the end you will have eliminated the confused-deputy god-credential.

## Background

This exercise covers material from:

- [Chapter 4 — OAuth, Tokens, and RBAC for Agent Toolchains](../04-oauth-rbac-tokens.md)
- [Chapter 3 — Least Privilege, Sandboxing, and Human Approval](../03-least-privilege-and-approval.md)

The system: a **workspace assistant** that, on behalf of a logged-in employee, integrates with three external APIs — a CRM (read/write contacts), a payments provider (issue refunds), and a calendar (read/write events). Two operating modes exist: **interactive** (employee present) and **scheduled** (a nightly digest agent, no human present). The deliverable is an architecture document with flow diagrams and the token-lifecycle design — design, not a running OAuth server.

## Prerequisites

- Ability to read OAuth 2.x flow diagrams.
- The RBAC and downscoping vocabulary from Chapter 4.

## Tasks

### 1. Choose the flow per integration and mode

- For each (integration × mode), pick the OAuth flow: **Authorization Code + PKCE**, **Client Credentials**, or **Token Exchange (downscoping)**. Justify each choice. The nightly digest cannot use a user-present flow — say what it uses and how you keep its scope tight.

### 2. Design the token lifecycle

- Specify, for access and refresh tokens: TTL, where each is stored, who can read it, how refresh happens out-of-band, and the rule that keeps tokens **out of the model's context**.
- Specify revocation and rotation: how a token is killed mid-session, and what triggers rotation.

### 3. Add audience binding and downscoping

- Show how a token issued to the agent is **audience-bound** to one resource and **downscoped per tool** (the CRM token cannot be replayed at the payments API).

### 4. Layer RBAC over the catalog

- Build the role → permitted-tools map. Define at least three roles (e.g. `sales_rep`, `finance`, `read_only`).
- Implement the per-call RBAC check, and show that **effective authority = RBAC ∩ OAuth scopes ∩ per-call policy**.

### 5. Trace one request end to end

- Pick "employee asks the agent to issue a $200 refund." Trace: identity → RBAC check → token acquisition/downscope → per-call policy → approval gate → API call. Show where each control can deny.

## Starter guidance

RBAC check + effective-authority intersection:

```python
ROLE_TOOLS = {
    "sales_rep":  {"crm_read", "crm_write", "calendar_read", "calendar_write"},
    "finance":    {"crm_read", "issue_refund"},
    "read_only":  {"crm_read", "calendar_read"},
}

def rbac_allows(roles: set[str], tool: str) -> bool:
    return any(tool in ROLE_TOOLS.get(r, set()) for r in roles)

def effective_allowed(tool: str, ctx) -> bool:
    return (
        rbac_allows(ctx.roles, tool)                 # role grants the tool
        and tool in ctx.token.scopes                 # OAuth token carries the scope
        and TOOL_POLICY[tool](ctx.call.arguments)    # per-call argument policy
    )
```

Token-lifecycle diagram:

```text
   employee ─consent(PKCE)─▶ Auth Server ──▶ access(TTL=10m, aud=crm) + refresh(secret store)
                                                  │
                            credential broker ◀───┘  (model never sees refresh token)
                                  │ token-exchange: downscope to {issue_refund, aud=payments}
                                  ▼
   tool middleware ──injects short-lived, audience-bound token at call time──▶ payments API
```

## Acceptance criteria

You can demonstrate that:

- Each integration × mode has a justified OAuth flow; the scheduled agent uses Client Credentials (or equivalent) with a tightly scoped service identity, not a god key.
- The token lifecycle specifies TTLs, out-of-band refresh, storage, revocation, rotation, and the no-token-in-context rule.
- Tokens are audience-bound and downscoped per tool.
- RBAC is defined for ≥3 roles and the per-call check enforces the RBAC ∩ scopes ∩ policy intersection.
- The end-to-end refund trace shows every point at which a control can deny.

## Reflection

In `NOTES.md`:

1. Where did the god-credential temptation appear in your design, and how did you remove it?
2. If an access token leaks via a log, what is the blast radius given your TTL and audience binding?
3. How does the nightly scheduled agent's lack of a human change your scope and approval design?

## Stretch goals

- Add **step-up authorization**: high-risk tools require a fresh, higher-assurance consent even within an active session.
- Design token-binding (DPoP / mTLS) so a stolen bearer token cannot be replayed from another client.
- Integrate with exercise-03's approval gate so `issue_refund` requires both RBAC `finance` and a human approval.
