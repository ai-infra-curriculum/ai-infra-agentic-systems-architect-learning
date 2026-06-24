# exercise-06: Excessive Agency Controls

**Estimated effort:** 4 hours

## Objective

Audit a given agentic system for **OWASP LLM06:2025 — Excessive Agency** and redesign it to close every gap. Excessive agency is damage caused not by a clever attack but by the agent simply *being able to do too much*: too many tools, too-broad permissions, too much autonomy. You will diagnose each of the three sub-failures, apply the matching control, and prove the redesigned system cannot take a high-impact action without a deterministic check or a human.

## Background

This exercise covers material from:

- [Chapter 3 — Least Privilege, Sandboxing, and Human Approval](../03-least-privilege-and-approval.md)
- [Chapter 1 — Guardrail Placement: Two-Sided Moderation and Per-Tool-Call Checks](../01-guardrail-placement.md)
- [Chapter 6 — Untrusted-Content Segregation and Adversarial Testing](../06-untrusted-content-and-red-teaming.md)

LLM06 decomposes into three sub-failures: **excessive functionality** (tools/permissions the task does not need), **excessive permissions** (tools that act with more authority than required), and **excessive autonomy** (the agent acts irreversibly without a human in the loop).

The system to audit: an **inbox-management agent** that triages a user's email. As shipped, it has tools `read_email`, `send_email`, `delete_email`, `archive_email`, `create_calendar_event`, `run_shell` (added "for automation"), and a single OAuth token with full mailbox + calendar + admin scope. It runs fully autonomously on a schedule. Deliverable: an audit report plus a redesigned permission/autonomy architecture — design with code stubs, not a running agent.

## Prerequisites

- The risk-tier and approval vocabulary from Chapter 3.
- Familiarity with OWASP LLM06:2025.

## Tasks

### 1. Audit excessive functionality

- List every tool the triage task does **not** need. Justify removal. (`run_shell` on an inbox agent is the obvious one — say what it should never have existed for.)

### 2. Audit excessive permissions

- Inspect the single full-scope token. Specify the **minimum** scopes triage actually requires and downscope to them. Identify which tools were acting with more authority than their function needs.

### 3. Audit excessive autonomy

- Identify every **irreversible** action the agent can take autonomously (`delete_email`, `send_email`). Move each behind a control: human approval, a reversible alternative (archive instead of delete), or a deterministic policy (only send to known contacts).

### 4. Redesign the architecture

- Produce the corrected tool catalog with per-tool risk tier, scope, and autonomy level (auto / policy-gated / human-approval).
- Implement the autonomy gate (see starter) that downgrades irreversible actions to proposals.

### 5. Prove containment

- For three "what if the model goes wrong" scenarios (hallucinated delete of all mail, injection-driven send to attacker, mass-unsubscribe loop), show the redesigned control that bounds each.

## Starter guidance

Three-axis audit table:

```text
| tool                  | needed? | min scope         | reversible? | autonomy level   |
|-----------------------|---------|-------------------|-------------|------------------|
| read_email            | yes     | mail.read         | n/a         | auto             |
| archive_email         | yes     | mail.modify       | yes         | auto             |
| create_calendar_event | yes     | calendar.events   | yes         | auto             |
| send_email            | yes     | mail.send         | NO          | human-approval   |
| delete_email          | replace | —                 | NO          | removed → archive|
| run_shell             | NO      | —                 | NO          | removed          |
```

Autonomy gate:

```python
from enum import Enum

class Autonomy(Enum):
    AUTO = "auto"                  # reversible, low-risk → just do it
    POLICY = "policy"              # allowed only if deterministic policy passes
    APPROVAL = "approval"          # irreversible → propose to human

def autonomy_gate(tool: str, args: dict, ctx) -> tuple[str, str]:
    spec = CATALOG[tool]
    if spec.autonomy is Autonomy.AUTO:
        return "ALLOW", "reversible/low-risk"
    if spec.autonomy is Autonomy.POLICY:
        ok = spec.policy(args, ctx)          # e.g. recipient in known contacts
        return ("ALLOW", "policy passed") if ok else ("DENY", "policy failed")
    return "REQUIRE_APPROVAL", spec.render(args)   # exact action shown to human
```

## Acceptance criteria

You can demonstrate that:

- All three LLM06 sub-failures are audited with concrete findings (excess functionality, permissions, autonomy).
- `run_shell` and any other unneeded tool are removed; the full-scope token is downscoped to minimum scopes.
- Every irreversible action is moved behind approval, a reversible alternative, or a deterministic policy.
- The autonomy gate downgrades irreversible actions to proposals and shows the exact action to a human.
- The three failure scenarios are each shown to be bounded by a named control.

## Reflection

In `NOTES.md`:

1. Which sub-failure (functionality, permissions, autonomy) was the most dangerous here, and why?
2. Where did "make it convenient" pressure push toward excessive agency, and how did you resist it without making the agent useless?
3. How does removing `delete_email` in favor of `archive_email` change the system's worst case?

## Stretch goals

- Add a **dry-run mode**: the agent produces its full proposed action plan for a human to approve as a batch before anything executes.
- Add anomaly detection on action volume (e.g. >20 archives/minute pauses for review).
- Write the adversarial suite (Chapter 6) covering all three sub-failures and wire it into CI so a future tool addition that reintroduces excessive agency fails the build.
