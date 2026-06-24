# Quiz — Evaluation Harnesses for Agentic Systems

Knowledge check for mod-304. Answer from the chapters, then expand each answer to self-check. Ten questions across trajectory vs. final-state eval, tool-call correctness, LLM-as-judge rubrics, and the offline/online gating strategy.

---

**1. An agent answers "the capital of France is Paris" without calling any tool. Final-state eval passes. What failure has final-state eval missed, and which lens catches it?**

<details>
<summary>Answer</summary>

The **right-answer-for-the-wrong-reason** failure: the agent reached a correct outcome via a fragile path (no retrieval), so it will fail the next question that genuinely needs retrieval. **Trajectory eval** — specifically a `retrieved_before_answering` predicate — catches it. Final-state eval cannot distinguish a robust strategy from a lucky guess.
</details>

---

**2. Why is asserting an agent's exact tool-call sequence usually a bad trajectory check? What should you assert instead?**

<details>
<summary>Answer</summary>

Exact-sequence assertions **over-specify**: a *better* trajectory that solves the task differently fails the eval, so you end up testing imitation of one reference path rather than whether the problem was solved. Assert **trajectory properties/predicates** instead (retrieved-before-answering, budget respected, no unconfirmed destructive call) — or, at most, an order-allowing superset match. Score outcomes strictly, paths loosely.
</details>

---

**3. Name the three flavors of trajectory match in increasing robustness, and when each is appropriate.**

<details>
<summary>Answer</summary>

1. **Exact match** — step-for-step; reserve for short deterministic flows where any deviation is a bug (e.g., auth-then-call).
2. **Order-allowing / superset match** — required steps appear in valid order, extra steps tolerated; good default for "must consult the policy doc before deciding."
3. **Property / predicate match** — a boolean function of the trajectory; the most robust and most-used, because it decouples "behaved correctly" from "reproduced a script."
</details>

---

**4. The two layers of a tool call are selection and arguments. A selection-layer failure and an argument-layer failure point you to different fixes — what are they?**

<details>
<summary>Answer</summary>

A **selection** failure (wrong tool picked) points to ambiguous/overlapping **tool descriptions or routing logic**. An **argument** failure (right tool, wrong call) points to the **parameter schema, parameter docs, or the model's extraction**. Lumping them into one "bad tool call" verdict throws away the diagnosis, which is why every check is tagged by layer.
</details>

---

**5. Schema validation passes on a tool call with `limit: 1000000` and a date range where `start > end`. Why isn't schema validity enough, and what check catches these?**

<details>
<summary>Answer</summary>

Schema-valid is not the same as **correct**: both values satisfy their types/formats but are operationally wrong. **Value-constraint checks** — deterministic predicates for bounds (`limit <= MAX`), cross-field invariants (`start <= end`), and task-consistency (an ID in the args matches the ID in the task) — catch them without a model.
</details>

---

**6. In the tool-call harness, the checks run as a cascade. What is the ordering, and what does the cascade buy you beyond correctness?**

<details>
<summary>Answer</summary>

Order: **schema validity → value constraints → faithfulness (LLM judge)**, cheapest deterministic checks first, short-circuiting on failure (never judge a call that already failed schema validation). Beyond a precise per-layer diagnosis, the cascade is the **cost control**: deterministic checks dispose of the vast majority of calls so the expensive judge runs only on the genuinely fuzzy minority.
</details>

---

**7. Why score an LLM-judge rubric on separate anchored axes (e.g., 0/1/2) rather than one "overall quality 1–10"?**

<details>
<summary>Answer</summary>

A blended 1–10 is **uninterpretable and unactionable** — you can't tell a fluent-but-wrong answer from a correct-but-incomplete one, and models can't reliably distinguish a 7 from an 8. **Separate axes** (accuracy, completeness, citation, tool efficiency) on a **short anchored discrete scale** give a vector you can threshold and diagnose independently, and the anchors make two runs of the judge mean the same thing by each score.
</details>

---

**8. What does it mean to "calibrate" an LLM judge, and why can't you skip it before gating on the judge?**

<details>
<summary>Answer</summary>

Calibration = **measuring the judge against a human-labeled gold set** (30–80 range-covering cases), reporting **per-axis agreement**, inspecting disagreements, fixing the rubric, and re-measuring — with human–human agreement as the ceiling you can't beat. You can't skip it because an uncalibrated judge produces authoritative-feeling but untrustworthy numbers; gating on an uncalibrated axis blocks good releases and waves through bad ones at random.
</details>

---

**9. Offline eval is green. Why is that necessary but not sufficient, and what does online eval add?**

<details>
<summary>Answer</summary>

Offline eval runs against a **fixed dataset** — a sample of a moving world — so it catches regressions but can't cover the long tail of real inputs or distribution shift. **Online eval** (shadow replay, canary, continuous reference-free scoring) sees the **real distribution** and catches what the dataset missed. Offline can't block on reality before users hit it; online can't block a bad build pre-deploy. You need both.
</details>

---

**10. Distinguish a relative gate from a hard gate in the release pipeline. Give an example where a build improves on one axis but must still be blocked.**

<details>
<summary>Answer</summary>

A **relative gate** blocks regressions versus the baseline (no axis drops more than ε) — it's comparative. A **hard gate** is an absolute floor that blocks **regardless of baseline**: any safety violation, unconfirmed destructive tool call, or schema-invalid rate above a tiny floor. Example: a candidate that **improves accuracy** but introduces **one unconfirmed destructive-tool call** must be blocked — the hard gate fires and does not trade off against the accuracy gain.
</details>

---

## Scoring

- **9–10 correct** — you can architect and defend an eval harness as a deployment gate.
- **6–8 correct** — solid; revisit the chapter behind any miss before [exercise-04](../exercises/exercise-04-eval-gated-release-pipeline.md).
- **< 6 correct** — re-read the chapters and re-run the quiz; the gating exercise composes all four.
