# Chapter 4 — OAuth, Tokens, and RBAC for Agent Toolchains

Least privilege (Chapter 3) decides *which tools* an agent may call. This chapter decides *as whom* it calls them and *with what credentials*. An agent toolchain reaches real systems — your CRM, a payments API, a cloud account — and those systems already have an authorization model. The architect's job is to connect the agent to them without the agent becoming a confused deputy that wields more authority than the user it serves.

## The confused-deputy trap

The naïve design gives the agent a single, powerful service credential — an admin API key baked into the deployment — and the agent acts with that authority for every user. Now a low-privilege user, via prompt injection or simply by asking, can make the agent do admin things. The agent is a *confused deputy*: it has high privilege and acts on low-privilege input. The fix is **delegated authorization**: the agent should act with the *user's* authority, scoped down, not with a god credential.

## OAuth 2.x for delegated tool access

OAuth is the standard answer to "let this software act on a user's behalf against that API, with a scoped, revocable, expiring grant." For agent toolchains, the relevant flows are:

- **Authorization Code + PKCE** — when a human user is present and consents. The user authorizes the agent to access a specific resource with specific scopes; the agent receives a short-lived **access token** and a **refresh token**. PKCE (Proof Key for Code Exchange) protects the exchange even for public clients.
- **Client Credentials** — for machine-to-machine, *no user present* (a scheduled agent). The agent authenticates as itself to obtain a token scoped to what that service account may do. Use this sparingly and scope it hard — it is the closest thing to the god credential.
- **Token Exchange (RFC 8693)** — to *downscope*: take the user's broad token and exchange it for a narrower one bound to the specific tool the agent is about to call.

```text
  user ──consent──▶ Authorization Server ──▶ access token (scopes: orders.read, refunds.write)
                                              + refresh token
        │                                         │
        ▼                                         ▼ (short TTL)
   ┌─────────┐   token (downscoped per tool)  ┌──────────────┐
   │  agent  │ ─────────────────────────────▶ │  resource API │
   └─────────┘   never the raw admin key      └──────────────┘
```

The principle: the token an agent carries to a tool should encode **exactly the scopes that tool needs, for this user, for a short window** — never a broad, long-lived credential.

## Token management is a lifecycle, not a variable

Tokens leak — through logs, through a hijacked agent, through an over-broad context window. Architect for that:

- **Short TTLs.** Access tokens live minutes, not days. A leaked token expires before most attacks complete.
- **Refresh out-of-band.** The refresh token lives in a secret store the agent process cannot freely read into its context; a credential broker exchanges it for fresh access tokens. The model never sees the refresh token.
- **Never in the prompt.** Tokens are injected by the tool-calling middleware at call time, not placed in the model's context. A token in the context is a token one injection away from exfiltration.
- **Revocation and rotation.** Every grant is revocable; assume any token may need to be killed mid-session. Rotate on suspicion.
- **Audience binding.** Bind tokens to the specific resource (`aud` claim) so a token for the email API cannot be replayed against the payments API.

## RBAC over the tool catalog

OAuth answers "can this agent reach this API." **RBAC** answers "is this *principal* allowed this *tool*." They compose: OAuth scopes the credential, RBAC scopes the catalog.

Model it as roles → permitted tools, and check it in the per-tool-call guard from Chapter 1:

```python
ROLE_TOOLS = {
    "support_agent":  {"get_order", "add_note", "create_draft"},
    "refund_officer": {"get_order", "issue_refund"},          # high-risk, still gated by approval
    "read_only":      {"get_order", "search_docs"},
}

def rbac_allows(principal_roles: set[str], tool: str) -> bool:
    return any(tool in ROLE_TOOLS.get(role, set()) for role in principal_roles)
```

The agent's effective authority is the **intersection** of (its RBAC-permitted tools) and (the OAuth scopes on its current token) and (the per-call argument policy). Three independent narrowings; an attacker has to defeat all three.

## Key takeaways

- Never give an agent a single god credential; that is a confused deputy waiting for an injection. Act with the user's downscoped authority.
- Use OAuth 2.x flows — Authorization Code + PKCE when a user consents, Client Credentials for unattended agents, Token Exchange to downscope per tool.
- Treat tokens as a lifecycle: short TTLs, out-of-band refresh, never in the model's context, revocable, audience-bound.
- RBAC scopes the tool catalog per principal; effective authority is the intersection of RBAC, OAuth scopes, and per-call policy.
