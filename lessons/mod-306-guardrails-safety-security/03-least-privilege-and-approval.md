# Chapter 3 — Least Privilege, Sandboxing, and Human Approval

Chapter 2 ended on a promise: even a fully hijacked agent should not be able to reach anything that matters. This chapter is how you keep that promise. The mechanism is least privilege applied to *tools* — the agent's hands — plus a human-in-the-loop gate on the actions you can never fully automate away.

## Least privilege for tools

The default failure is over-provisioning: someone gives the agent a `database` tool that can run arbitrary SQL, a `shell` tool that can run any command, and an `http` tool that can reach any host. Now a single successful injection owns your data plane. Least privilege inverts this — grant the *narrowest capability that completes the task*, and nothing more.

Concretely, prefer many narrow tools over one broad one:

- Not `run_sql(query)` but `get_order(order_id)`, `list_orders_for_user(user_id)`. The narrow tool cannot be coerced into `DROP TABLE`.
- Not `http_request(url)` but `fetch_doc(doc_id)` resolving against an internal catalog. The narrow tool cannot reach `evil.example`.
- Not `run_shell(cmd)` but `run_tests()`, `format_code()`. The narrow tool has no `curl | sh`.

Every tool gets a written **capability spec**: what it can touch, its argument policy envelope, its risk tier, and the role required to call it. This spec is the input to the per-tool-call guard from Chapter 1.

```python
@dataclass(frozen=True)
class ToolSpec:
    name: str
    required_role: str          # RBAC gate (Chapter 4)
    risk: Risk                  # LOW | MEDIUM | HIGH
    args_within_policy: Callable[[dict], bool]   # e.g. amount <= cap, path in allowlist
    side_effects: bool          # read-only tools are inherently lower risk
```

## The risk tiers and the approval gate

Not every action carries the same blast radius. Classify each tool into a tier and let the tier drive the control:

| Tier | Examples | Control |
|------|----------|---------|
| **Read-only** | `get_order`, `search_docs` | Allow; log. Cannot mutate state. |
| **Reversible write** | `add_note`, `create_draft` | Allow within policy; log; rate-limit. |
| **Irreversible / high-value** | `issue_refund`, `delete_account`, `send_email`, `wire_transfer` | **Require human approval.** |

The high tier is where **human-in-the-loop** lives. The agent does not perform the action; it *proposes* it, and a person with the authority and context approves or rejects. Crucially, the approval surfaces the *concrete action and arguments*, not a vague "the agent wants to do something" — the approver needs to see "refund $4,210 to account X" so they can catch the injected one.

```python
def gate(call: ToolCall, spec: ToolSpec, ctx) -> Decision:
    if spec.risk is Risk.HIGH:
        return Decision.REQUIRE_APPROVAL(
            summary=spec.render_for_human(call.arguments),   # human-readable, exact
            approver_role=spec.required_role,
        )
    return Decision.ALLOW if spec.args_within_policy(call.arguments) else Decision.DENY
```

Design the approval so it cannot be defeated by volume — if the agent can queue 500 refund approvals, an approver fatigues and rubber-stamps. Rate-limit proposals, batch related ones, and make rejection the low-friction default.

## Sandboxing as the privilege boundary for code

When a tool *runs code* (a `shell`, a `python_exec`, a CLI agent), the capability spec is not enough — the code can do anything the process can. Here least privilege means **sandboxing**: the code runs in a container or VM with no credentials mounted, a read-only or scratch filesystem, no network except an explicit egress allowlist, and CPU/memory/time limits. Chapter 5 designs this in depth; the principle here is that the *sandbox boundary* is the privilege boundary, and it must be enforced by the OS/runtime, not by asking the model nicely.

## Key takeaways

- Grant the narrowest tool that completes the task; prefer many specific tools over one general one, so a hijacked agent has nothing dangerous to reach.
- Every tool carries a capability spec — touchable resources, argument policy, risk tier, required role — that feeds the deterministic per-call guard.
- Tier actions by blast radius; read-only flows freely, irreversible/high-value actions require a human approval that shows the *exact* action and arguments.
- When a tool runs code, the privilege boundary is the sandbox enforced by the runtime, not a prompt instruction.
