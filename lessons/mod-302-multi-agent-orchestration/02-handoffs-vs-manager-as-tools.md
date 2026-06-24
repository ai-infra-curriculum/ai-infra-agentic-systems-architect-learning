# Chapter 2 — Handoffs vs. Manager-as-Tools, and Answer Ownership

Once you've decided a task needs more than one agent, the next architectural fork is *how delegation works*. There are two dominant styles, and they differ on a question that is easy to overlook and expensive to get wrong: **who owns the final, user-facing answer?**

## The two delegation styles

**Handoff (control transfer).** Control *moves* to a specialist. Agent A decides the request belongs to Agent B, hands over the conversation, and B now drives — B talks to the user directly. A is out of the loop unless B hands back.

```text
  user ──▶ triage agent ──handoff──▶ refunds agent ──▶ user
                  │                        (refunds owns the answer)
                  └─ no longer in the loop
```

**Manager-as-tools.** A **manager** agent stays in control and calls specialists as if they were tools. The specialists return results *to the manager*, which composes the final answer. Specialists never speak to the user.

```text
  user ──▶ manager ──┬─call──▶ specialist A ──return──┐
                     ├─call──▶ specialist B ──return──┤
                     └────────────────────────────────┘
              manager composes ──▶ user
                  (manager owns the answer)
```

## The ownership question

The single decision that organizes everything else: **which agent's voice reaches the user, and who is accountable for the final answer's correctness, tone, and safety?**

| | Handoff | Manager-as-tools |
| --- | --- | --- |
| Final answer owned by | The agent currently holding control | The manager, always |
| User-facing voice | Whichever specialist is active | Single, consistent (the manager) |
| Specialist autonomy | High — drives the conversation | Low — returns a result, then yields |
| Context cost | Lower — one active agent at a time | Higher — manager re-pays specialist returns in its context |
| Guardrail/safety choke point | Diffuse — every specialist must enforce | Single — enforce at the manager |
| Best when | Specialists need deep, multi-turn ownership of a sub-domain | You need one consistent voice and a single accountability point |

## Choosing

Pick **handoffs** when sub-domains are genuinely distinct and a specialist needs to *own a multi-turn interaction* — a refunds agent that must walk a user through a multi-step process is better off holding the conversation than being poked tool-style on every turn. The cost: your safety and tone guarantees now have to hold in *every* specialist, because any of them can be the voice the user hears.

Pick **manager-as-tools** when you need **one consistent voice, one accountability point, and one place to enforce guardrails**. The manager is the only thing the user hears, so you reason about correctness and safety in one spot. The cost: the manager re-pays every specialist's return in its own context (it leans toward the higher end of the token multiplier from Chapter 1), and specialists can't carry a long sub-conversation on their own.

A common architecture blends them: a manager owns the user-facing answer, but for a deep sub-domain it *hands off* to a specialist that owns that slice of the conversation and hands back when done. Draw the handback edge explicitly — an implicit one is how conversations get stranded.

## Failure modes to design against

- **Handoff loops / ping-pong.** A → B → A → B with no progress. Bound transfers (Chapter 4) and require each handoff to carry *why* and *what's needed*.
- **Orphaned ownership.** After a handoff, no agent believes it owns the answer and the user is left hanging. Every handoff must name the new owner.
- **Diffuse guardrails.** With handoffs, a safety rule enforced only in the manager is silently bypassed once control transfers. Either centralize on manager-as-tools, or replicate the guardrail into every specialist.
- **Lost context on transfer.** A handoff that drops the reason for transfer forces the receiver to re-derive intent. Make the transfer payload a typed contract: `{reason, user_intent, state}`.

## Key takeaways

- Two delegation styles: **handoff** transfers control (the receiver owns the answer and the voice); **manager-as-tools** keeps the manager in control (it always owns the answer).
- The organizing decision is **answer ownership** — it determines the user-facing voice, the accountability point, and where guardrails live.
- Handoffs give specialists deep autonomy at the cost of diffuse safety enforcement; manager-as-tools gives one voice and one choke point at the cost of higher context spend.
- Blends are common and fine — but draw the handback edge and name the new owner on every transfer, or you get loops and orphaned answers.
