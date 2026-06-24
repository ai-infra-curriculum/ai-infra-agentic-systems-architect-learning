# exercise-03: Least-Privilege Tool Permissions

**Estimated effort:** 3 hours

## Objective

Design a **least-privilege permission architecture** for an agent's tool catalog: refactor broad tools into narrow ones, build a risk-tiered permission matrix, and wire a tiered approval gate so that irreversible actions require a human who sees the exact action. By the end you will have turned "the agent can do anything" into "a hijacked agent can reach nothing dangerous."

## Background

This exercise covers material from:

- [Chapter 3 — Least Privilege, Sandboxing, and Human Approval](../03-least-privilege-and-approval.md)
- [Chapter 1 — Guardrail Placement: Two-Sided Moderation and Per-Tool-Call Checks](../01-guardrail-placement.md)

The starting point is a deliberately over-provisioned **DevOps assistant**: it currently has `run_sql(query)`, `http_request(url, method, body)`, and `run_shell(cmd)`. Your job is to redesign its tool surface for least privilege.

## Prerequisites

- The capability-spec and risk-tier vocabulary from Chapter 3.
- Comfort writing argument-policy predicates.

## Tasks

### 1. Diagnose the over-provisioning

- For each of the three broad tools, name one destructive action and one exfiltration action a single prompt injection could trigger today.

### 2. Refactor into narrow tools

- Replace each broad tool with a set of **narrow, intent-specific** tools (e.g. `run_sql` → `get_deploy_status(service)`, `list_recent_errors(service, since)`). Each narrow tool can do exactly one safe thing.
- State which previously-possible destructive/exfiltration actions are now *structurally impossible* (no tool exists to do them).

### 3. Build the permission matrix

- Write a capability spec per narrow tool: risk tier, required role, argument policy envelope, side-effect flag, and approval requirement.

### 4. Wire the tiered approval gate

- Implement `gate(call, spec, ctx)` (see starter) that allows read-only and reversible actions within policy, and returns `REQUIRE_APPROVAL` with a **human-readable rendering of the exact action** for the irreversible tier.

### 5. Defend against approval fatigue

- Add a mechanism (rate limit, batching, or dedup) so the agent cannot flood approvers. Describe the policy and why it prevents rubber-stamping.

## Starter guidance

Capability spec + gate:

```python
from dataclasses import dataclass
from enum import Enum
from typing import Callable

class Risk(Enum):
    LOW = "low"          # read-only
    MEDIUM = "medium"    # reversible write
    HIGH = "high"        # irreversible / high-value

@dataclass(frozen=True)
class ToolSpec:
    name: str
    required_role: str
    risk: Risk
    args_within_policy: Callable[[dict], bool]
    render_for_human: Callable[[dict], str]   # exact action, e.g. "DELETE service api-prod"

class Decision(Enum):
    ALLOW = "allow"; DENY = "deny"; REQUIRE_APPROVAL = "require_approval"

def gate(call, spec: ToolSpec, ctx) -> tuple[Decision, str]:
    if spec.required_role not in ctx.roles:
        return Decision.DENY, "role not granted"
    if not spec.args_within_policy(call.arguments):
        return Decision.DENY, "arguments outside policy envelope"
    if spec.risk is Risk.HIGH:
        return Decision.REQUIRE_APPROVAL, spec.render_for_human(call.arguments)
    return Decision.ALLOW, "within policy"
```

Permission-matrix skeleton:

```text
| tool                | tier   | required_role | arg policy                  | approval |
|---------------------|--------|---------------|-----------------------------|----------|
| get_deploy_status   | LOW    | dev           | service in catalog          | no       |
| list_recent_errors  | LOW    | dev           | since <= 24h                | no       |
| restart_service     | MEDIUM | sre           | service in catalog          | no       |
| scale_service       | MEDIUM | sre           | replicas <= 20              | no       |
| delete_service      | HIGH   | sre_lead      | service in catalog          | YES      |
```

## Acceptance criteria

You can demonstrate that:

- Each broad tool is replaced by narrow tools, and you list destructive/exfiltration actions that are now structurally impossible.
- Every narrow tool has a complete capability spec with risk tier, role, argument policy, and approval flag.
- `gate` is deterministic and returns `REQUIRE_APPROVAL` with an exact human-readable action for the HIGH tier.
- The approval-fatigue mechanism is specified and prevents flooding.
- No remaining tool is a general-purpose escape hatch (`run_shell`, `run_sql`, `http_request`).

## Reflection

In `NOTES.md`:

1. Which narrow tools did you have to add that you did not expect, and what does that say about the original task surface?
2. Is there any task the broad tools could do that no combination of narrow tools can? Is that loss acceptable?
3. How does this matrix become the input to the RBAC layer in exercise-04?

## Stretch goals

- Add a **break-glass** path: a documented, audited way to grant a broad capability in an incident, with mandatory dual-approval and auto-expiry.
- Add per-tool rate limits and argument-value anomaly detection (e.g. `scale_service` to 20 replicas at 3 a.m.).
- Write the adversarial tests that prove a hijacked agent cannot reach any HIGH-tier action without approval.
