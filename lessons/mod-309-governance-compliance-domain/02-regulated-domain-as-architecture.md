# Chapter 2 — Regulated Domain as Architecture

[Chapter 1](01-horizontal-governance-frameworks.md) gave you the horizontal frame — the frameworks that apply to any AI system. This chapter gives you the *translation method*: how to turn a regulatory constraint into a piece of architecture, **independent of which statute it came from.**

The temptation, when a regulation lands, is to read the statute and ask "what does HIPAA require?" or "what does PCI require?" That framing traps you in one sector. The architect's framing is different: **almost every regulation, in every sector, is built out of the same five primitive constraints.** Privacy, residency, auditability, accountability, and retention recur across HIPAA, GLBA, SOX, PCI, FERPA, COPPA, GDPR, and public-sector records law — with different thresholds and vocabulary, but the *same architectural shape*. Learn to translate the five primitives into components once, and you can satisfy any regime by setting their parameters, rather than re-deriving the architecture each time.

> **Statute-independent by design.** This chapter deliberately names *no* sector as the default. A "data subject" might be a patient, a borrower, a citizen, or a student; "the record" might be a medical chart, a transaction, a case file, or a grade. The architecture is the same. The sector only sets the dials — which we do in [Chapter 3](03-multi-regime-mapping.md).

## The five primitive constraints

Strip the legalese off almost any data regulation and you find some combination of these five. They are the architect's universal decomposition.

| Primitive | The constraint, generically | The architectural question it forces |
| --- | --- | --- |
| **Privacy** | Who/what may see, use, or share which data, for which purpose. | Where are the data-flow boundaries, and what minimization happens at each? |
| **Residency** | Where data may physically or legally live and be processed. | Which components are region-pinned, and where does data cross a boundary? |
| **Auditability** | What must be provably recorded about what happened. | What is logged, where, immutably, and for how long is it queryable? |
| **Accountability** | Who is the responsible human/role for a decision or action. | Where are the named-owner decision points and human gates? |
| **Retention** | How long data may/must be kept, and how it must be destroyed. | What is the lifecycle of each data class — TTL, archival, deletion? |

Every regulation is, architecturally, a *specific setting* of these five plus some sector-specific extras. The translation method is: **(1) decompose the regulation into these primitives, (2) translate each primitive into a concrete architectural element, (3) parameterize that element with the regime's specific thresholds.** The rest of this chapter walks each primitive and the architecture it produces.

## Privacy → data-flow boundaries and minimization

Privacy constraints are fundamentally about *which data crosses which boundary, and why.* In an agentic system the data flow is wide: user input, retrieved context, tool inputs and outputs, model prompts and completions, logs, and traces. A privacy constraint resolves into **explicit boundaries** in that flow, with a control at each:

```text
   user / data source
        │  (1) ingest boundary — classify & tag data sensitivity
        ▼
   ┌─────────────┐   (2) minimization — strip/mask/tokenize before it
   │  agent core │       reaches the model or a tool
   └─────┬───────┘
         │  (3) tool boundary — what may leave to each tool/extension
         ▼
   tools / extensions / sub-agents
         │  (4) egress boundary — what may leave the trust domain
         ▼
   logs / traces / external systems
```

The architectural elements a privacy constraint produces:

- **Data classification at ingest.** Tag every datum with a sensitivity class (e.g., "identifiable," "sensitive-category," "public"). The class drives every downstream control. Without classification, no other privacy control can be enforced.
- **Minimization before exposure.** Strip, mask, tokenize, or redact identifiable data *before* it reaches a component that does not need it — especially before it enters a model prompt, a tool call, or a log. The principle ("collect and expose the minimum necessary") is statute-independent; the *threshold* (what counts as identifiable) is the regime parameter.
- **Purpose binding.** A datum collected for one purpose may not silently flow to another. Architecturally this is access policy on the data, scoped by purpose — and it is why an agent's tools must be *scoped* (mod-306) rather than handed the whole context.
- **Boundary on tools and sub-agents.** Every tool and extension is a potential egress point. What context an agent passes to a tool is a privacy decision, governed by the lifecycle of [Chapter 4](04-extension-and-tool-governance.md).

## Residency → region-pinned components and crossing controls

Residency constraints say *data may only live or be processed in certain places.* This is increasingly common and bites agentic systems hard, because the model inference, the vector store, the tool backends, and the logs may each sit in different regions or clouds.

The architecture this produces:

- **Region-pinned data planes.** Storage (vector DB, document store, run state) and processing (inference endpoint, tool execution) are pinned to allowed regions. The architect's artifact is a **data-residency map**: for each data class, which region(s) it may live and be processed in.
- **Crossing controls.** Wherever data *must* cross a boundary, there is an explicit, logged, and minimized crossing — never an accidental one. A model endpoint in another region is a residency event; so is a log shipped to a central store.
- **Per-region deployment topology.** When a regime forbids crossing, the answer is often a *per-region deployment* of the whole agent stack, with run state and retrieval kept local. This is an architecture decision with real cost and operational weight — exactly the kind the architect owns.

The trap: teams treat residency as "pick a region for the database" and forget the model inference, the third-party tool calls, and the trace pipeline. **Every component that touches regulated data is in scope for residency, including the LLM call and the observability stack.**

## Auditability → immutable, queryable logs that are real

Auditability is the constraint that what happened can be *proven*, after the fact, to someone who was not there. For an agentic system this is demanding, because "what happened" includes a non-deterministic model decision, a chain of tool calls, and possibly a human approval — across a run that may have spanned days.

Auditability is **not** the same as ordinary application logging. The architectural requirements are sharper:

- **Immutability / tamper-evidence.** An audit log a privileged user can quietly edit is not an audit log. The store is append-only or tamper-evident (write-once, hash-chained, or externally anchored). This rides naturally on the durable event history of [mod-308](../mod-308-deployment-durable-execution/README.md), which is already an immutable record of what a run did.
- **Completeness over the action surface.** The log must capture the *decisions and actions*, not just errors: what the agent proposed, what tool it called with what (minimized) arguments, what a human decided at a gate, and why. This is where the traces of [mod-305](../mod-305-observability-tracing/README.md) become governance evidence.
- **Queryability and retention.** An auditor asks "show me every action this agent took on this data subject in this period." The log must answer that within the regime's required retention window — which ties auditability to retention (below).
- **Privacy-aware logging.** Logs are themselves a data flow, subject to the privacy and residency constraints above. Logging raw identifiable data to satisfy auditability while *violating* privacy is a common own-goal; minimize in the log, too.

Auditability is the primitive most often claimed and least often real. The test: *can you actually produce, on demand, a tamper-evident record of what a specific run did and who approved it?* If not, the control is aspirational.

## Accountability → named owners and human decision points

Accountability answers *who is responsible.* Regulations increasingly demand a **named human** accountable for a decision — not "the AI decided." For an autonomous agent fleet this is the hardest primitive, because the whole point of autonomy is that no human was in the specific loop.

The architecture this produces:

- **Human-accountability points.** Specific places in the flow where a named human/role takes responsibility — the durable HITL gates of [mod-308](../mod-308-deployment-durable-execution/README.md), placed by *consequence*. A high-risk decision must have an accountable human at or after it.
- **Decision records with owners.** Consequential automated decisions carry a record naming the responsible role, the basis for the decision, and (where required) a path to contest or override it. This is the EU AI Act's "human oversight" obligation made concrete.
- **A RACI for the fleet.** At the platform level, accountability is an explicit mapping of who is Responsible/Accountable/Consulted/Informed for the agents' behavior — the subject of [Chapter 5](05-fleet-governance-and-accountability-specs.md).

Accountability is where governance stops being technical. The architect's job is to ensure the architecture *makes* accountability assignable — that there is a place to attach a name — not to assign the names personally.

## Retention → a lifecycle per data class

Retention is the lifecycle constraint: data must be kept for *at least* some period (for audit) and *no longer than* some period (for privacy), then destroyed in a specified way. These two pressures often conflict, which is why retention is an architecture decision, not a config flag.

The architecture this produces:

- **A retention schedule per data class.** Each class of data (input, retrieved context, prompts/completions, logs, run state, audit records) gets an explicit minimum and maximum lifetime and a destruction method. The audit record may need to outlive the raw data it describes — store them separately with separate schedules.
- **Automated lifecycle enforcement.** TTLs and deletion jobs that actually fire, not a policy in a doc. An agent's memory ([mod-303](../mod-303-memory-context-architecture/README.md)) and trace stores are common places where data quietly accumulates past its retention limit.
- **Right-to-erasure handling.** Where a regime grants deletion rights, the architecture must be able to *find and remove* a data subject's data across the agent's memory, vector store, logs, and run state — which is only possible if the data was classified and tagged at ingest. Erasure is where the privacy primitive's classification pays off.

## The translation method, end to end

Putting it together, the architect's repeatable procedure for any regulation:

```text
   1. DECOMPOSE the regulation into the five primitives
      (privacy / residency / auditability / accountability / retention)
      + note any sector-specific extras

   2. TRANSLATE each primitive into its architectural element
      privacy        → data-flow boundaries + minimization + classification
      residency      → region-pinned components + crossing controls
      auditability   → immutable, queryable, minimized logs
      accountability → human-accountability points + decision records + RACI
      retention      → per-class lifecycle (min/max + destruction)

   3. PARAMETERIZE each element with the regime's thresholds
      (what counts as identifiable, which regions, how long, who must approve)

   4. VERIFY each element resolves to a real component a reviewer can inspect
```

The payoff is reuse. Because steps 1–2 are statute-independent, the *same architecture* serves every regime; only step 3 changes per regime. That is exactly what makes the multi-regime mapping of [Chapter 3](03-multi-regime-mapping.md) tractable: you are not building four architectures, you are setting four parameter profiles on one.

## Worked judgment

*A diagnostics-support agent (healthcare).* Decompose: **privacy** (patient identifiers must be minimized before they reach the model and tools) → classification + masking boundary; **residency** (some jurisdictions require in-country processing) → region-pinned inference and store; **auditability** (every clinical recommendation must be reconstructable) → tamper-evident decision log on the durable history; **accountability** (a clinician, not the agent, is responsible) → HITL gate with a named clinician before any clinical action; **retention** (medical records have long mandated retention) → long-lifetime audit store, shorter-lifetime prompt cache. Notice: *no HIPAA clause numbers were needed to derive the architecture* — only the primitives. The clause numbers come in at step 3 to set the thresholds.

*A transaction-monitoring agent (finance).* The *same five primitives* produce a recognizably similar architecture — minimization at ingest, region-pinned processing, immutable decision logs, an accountable analyst at the consequential gate, per-class retention — with different parameters (PCI sets card-data handling, SOX sets audit-record integrity and retention, GLBA sets privacy scope). The structural overlap with the healthcare case is the whole point: it is the convergence you will formalize next chapter.

## Key takeaways

- A regulated domain is **architecture, not paperwork**: almost every regulation decomposes into five primitives — **privacy, residency, auditability, accountability, retention** — that recur across every sector.
- Each primitive translates into a concrete element: privacy → **data-flow boundaries + minimization + classification**; residency → **region-pinned components + crossing controls**; auditability → **immutable, queryable, minimized logs**; accountability → **human-accountability points + decision records + RACI**; retention → **per-class lifecycle**.
- The translation method — **decompose → translate → parameterize → verify** — is statute-independent. Steps 1–2 are reusable; only step 3 (the thresholds) changes per regime, which is what makes one architecture serve many regimes.
- Auditability and accountability are the primitives most often *claimed* and least often *real*. The test for each control is the same as the whole module's: can a reviewer point at the component, the log, or the named owner? If not, you have not finished translating.
