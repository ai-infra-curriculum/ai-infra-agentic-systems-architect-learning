# Chapter 1 — Guardrail Placement: Two-Sided Moderation and Per-Tool-Call Checks

A guardrail is not a feature you add; it is a component you *place*. The same content filter is worthless on the wrong edge and decisive on the right one. The architect's job is to draw the request path and mark every point where untrusted data crosses a trust boundary — and put a check there.

A useful agent has at least three such boundaries: input enters the system, the model decides to call a tool, and output leaves the system. Each gets its own guardrail, with its own failure mode and its own owner.

```text
            ┌──────────────┐         ┌──────────────┐         ┌──────────────┐
 user ────▶ │ INPUT guard  │ ──────▶ │  agent loop  │ ──────▶ │ OUTPUT guard │ ───▶ user
 input      │ (pre-model)  │         │  (model +    │         │ (post-model) │
            └──────────────┘         │   tools)     │         └──────────────┘
                                     └──────┬───────┘
                                            │ tool call
                                            ▼
                                     ┌──────────────┐
                                     │ PER-TOOL-CALL │  ← the guard that actually
                                     │  guard        │     stops damage
                                     └──────────────┘
```

## Two-sided moderation

**Input moderation** runs before the model sees the request. It screens for policy-violating content, obvious jailbreak strings, and PII you are not allowed to process. Its job is *triage*, not security — a determined attacker will paraphrase past any input classifier, so do not treat it as your prompt-injection defense (that is Chapter 2). Treat it as the cheap first filter that keeps junk out of your token budget and catches the unsophisticated 80%.

**Output moderation** runs after the model produces a response and before it reaches the user or a downstream system. It checks for leaked secrets, disallowed content, and PII the model may have surfaced from a tool result. Output moderation is more load-bearing than input moderation because it is your last line before damage becomes visible.

Run both as deny-by-exception, not allow-by-exception: a guardrail that *fails open* (passes traffic when the classifier errors or times out) is a guardrail an attacker will simply DoS. Decide the failure posture explicitly per guardrail and write it down.

## The per-tool-call guardrail is the one that matters

Input and output guards see text. The per-tool-call guard sees *intent that is about to become an action* — the exact place where prompt injection turns into damage. This is where you enforce the rules that can never be argued past by clever phrasing:

```python
def authorize_tool_call(call: ToolCall, ctx: RequestContext) -> Decision:
    spec = TOOL_CATALOG[call.name]
    # 1. Is this principal even allowed this tool? (RBAC — Chapter 4)
    if spec.required_role not in ctx.principal.roles:
        return Decision.DENY("role not granted")
    # 2. Are the arguments inside policy? (e.g. amount caps, path allowlist)
    if not spec.args_within_policy(call.arguments):
        return Decision.DENY("arguments outside policy envelope")
    # 3. Is this a high-risk action that needs a human? (Chapter 3)
    if spec.risk == Risk.HIGH:
        return Decision.REQUIRE_APPROVAL(call)
    return Decision.ALLOW
```

Notice what this is *not*: it is not the model deciding whether it is allowed to act. The authorization runs in deterministic code, outside the model's context, on arguments the model produced. The model can be fully convinced by an injected instruction that it should wire $1M offshore — and the per-tool-call guard, which never read the malicious document, denies it on the amount cap.

## Layering and the trust gradient

Guardrails compose along a gradient: cheap and probabilistic at the edges (classifiers, regex), expensive and deterministic at the action boundary (policy engine, RBAC, approval). Never let a single guardrail carry the whole load — that is the single-point-of-failure that incident reviews are made of. Defense in depth means an attacker has to defeat the input filter *and* the per-tool-call policy *and* the output filter, and the deterministic ones in that chain cannot be talked out of their decision.

## Key takeaways

- A guardrail is placed on a boundary, not bolted onto a model; draw the request path first, then mark every trust crossing.
- Two-sided moderation (input triage + output last-line) plus a deterministic per-tool-call check is the minimum viable guardrail set.
- The per-tool-call guard is load-bearing: it runs in code, outside the model's context, and converts "the model was convinced" into "the action was still denied."
- Decide each guardrail's fail-open vs. fail-closed posture explicitly; an undocumented failure mode is the one attackers find.
