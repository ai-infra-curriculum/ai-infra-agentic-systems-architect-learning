# exercise-01: Guardrail Placement Architecture

**Estimated effort:** 3 hours

## Objective

Take a realistic agentic system and produce its **guardrail-placement architecture**: a request-path diagram with input moderation, output moderation, and per-tool-call guardrails placed on the correct trust boundaries, each with a documented job, failure posture, and owner. By the end you will be able to defend *why* each guardrail sits where it does — and why a single content filter is not an architecture.

## Background

This exercise covers material from:

- [Chapter 1 — Guardrail Placement: Two-Sided Moderation and Per-Tool-Call Checks](../01-guardrail-placement.md)
- [Chapter 3 — Least Privilege, Sandboxing, and Human Approval](../03-least-privilege-and-approval.md)

The system to secure: a **customer-support agent** for an e-commerce company. It can `get_order(order_id)`, `search_docs(query)`, `add_note(order_id, text)`, `create_email_draft(to, body)`, and `issue_refund(order_id, amount)`. It answers customer questions in a chat widget and retrieves from a knowledge base. This is a design exercise — the deliverable is diagrams and a written justification, not a running agent (one guardrail stub is included).

## Prerequisites

- Comfort reading a data-flow diagram and marking trust boundaries.
- The tool/risk-tier vocabulary from Chapter 3.

## Tasks

### 1. Draw the request path

- Produce a `text` diagram of the full request path from customer input to response, including the agent loop, each tool, and the knowledge-base retrieval.
- Mark every point where data crosses a **trust boundary** (untrusted input in, retrieved content in, action out, response out).

### 2. Place the guardrails

- Place **input moderation**, **output moderation**, and a **per-tool-call guardrail** on the diagram at the correct boundaries.
- For each guardrail, write a one-paragraph spec: its job, what it checks, its **fail-open vs. fail-closed** posture (with justification), and which team owns it.

### 3. Build the per-tool-call policy table

- For each of the five tools, record: risk tier (read-only / reversible / irreversible), required role, argument policy envelope (e.g. `issue_refund.amount <= order_total`), and whether it requires human approval.

### 4. Justify the layering

- Write a short defense of why two-sided moderation plus a per-tool-call guard beats a single input filter. Give one concrete attack that the single filter misses and the per-tool-call guard catches.

### 5. Implement the per-tool-call guard

- Implement `authorize_tool_call` for this catalog (see starter). It must run in deterministic code, consult the policy table, and return `ALLOW`, `DENY(reason)`, or `REQUIRE_APPROVAL`.

## Starter guidance

Permission-matrix skeleton:

```text
| tool               | tier         | required_role  | arg policy                | approval |
|--------------------|--------------|----------------|---------------------------|----------|
| get_order          | read-only    | support_agent  | order_id owned by user    | no       |
| search_docs        | read-only    | support_agent  | —                         | no       |
| add_note           | reversible   | support_agent  | len(text) <= 2000         | no       |
| create_email_draft | reversible   | support_agent  | to in customer domain     | no       |
| issue_refund       | irreversible | refund_officer | amount <= order_total     | YES      |
```

```python
from dataclasses import dataclass
from enum import Enum

class Decision(Enum):
    ALLOW = "allow"
    DENY = "deny"
    REQUIRE_APPROVAL = "require_approval"

@dataclass(frozen=True)
class ToolCall:
    name: str
    arguments: dict

def authorize_tool_call(call: ToolCall, ctx) -> tuple[Decision, str]:
    spec = TOOL_CATALOG[call.name]          # from your permission table
    if spec["required_role"] not in ctx.roles:
        return Decision.DENY, "role not granted"
    if not spec["args_ok"](call.arguments, ctx):
        return Decision.DENY, "arguments outside policy envelope"
    if spec["approval"]:
        return Decision.REQUIRE_APPROVAL, "high-risk action"
    return Decision.ALLOW, "within policy"
```

## Acceptance criteria

You can demonstrate that:

- The diagram shows the request path with every trust boundary explicitly marked.
- Input, output, and per-tool-call guardrails are each placed and specced, including an explicit fail-open/fail-closed decision per guardrail.
- The per-tool-call policy table is complete for all five tools, with `issue_refund` requiring approval.
- `authorize_tool_call` is deterministic, runs outside the model's reasoning, and correctly returns `DENY` / `REQUIRE_APPROVAL` for the relevant cases.
- Your layering defense names a concrete attack the single input filter misses.

## Reflection

In `NOTES.md`:

1. Which guardrail did you make fail-closed, and what is the availability cost of that choice?
2. Where would a single content filter have given a false sense of security in this system?
3. If the company added a `cancel_subscription` tool, where does it land in your table and why?

## Stretch goals

- Add a fourth guardrail: a **retrieval guardrail** that tags knowledge-base results as untrusted before they reach the model.
- Add rate limiting to the approval path and describe how it defeats approval fatigue.
- Extend `authorize_tool_call` to emit a structured audit log line per decision for the observability module.
