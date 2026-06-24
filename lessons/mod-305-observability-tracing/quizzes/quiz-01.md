# mod-305 Knowledge Check — Observability & Tracing for Agents

**Total questions:** 10

**Passing score:** 70% (7/10)

Covers span conventions, agent-vs-APM signals, tracing non-deterministic agents, and platform trade-offs. Answer all questions, then check against the key at the end.

---

### Question 1: GenAI operation names

Under the OpenTelemetry GenAI semantic conventions, which span operation represents **one agent's invocation** in a multi-agent run?

A) `chat`
B) `execute_tool`
C) `invoke_agent`
D) `text_completion`

---

### Question 2: Token attributes

On which span type do you record `gen_ai.usage.input_tokens` and `gen_ai.usage.output_tokens`?

A) The `invoke_agent` span only
B) The `chat` (model inference) span
C) The `execute_tool` span
D) The root trace, never a span

---

### Question 3: Span nesting

In the OTel Python SDK, what makes a `chat` span created inside a worker automatically attach as a child of that worker's `invoke_agent` span?

A) Passing the parent span object into every function
B) Context propagation — the `invoke_agent` span is the current span when the `chat` span is created
C) Sharing the same `trace_id` string manually
D) A `parent_span_id` attribute you set by hand

---

### Question 4: Resource attributes

Why do `service.version` and `deployment.environment` belong on the OTel `Resource` rather than being set per-span?

A) They are required by the OTLP wire format
B) They are set once per process and stamped on every span, which is what lets you slice quality by deploy later
C) Spans cannot carry string attributes
D) They reduce the trace's total token count

---

### Question 5: Agent observability vs. APM

A run returns a `200` in 700ms but its answer cites a source it never retrieved. How do the two observability planes classify it?

A) Both APM and the quality plane mark it a failure
B) APM marks it a success; the quality plane (faithfulness) marks it a failure
C) APM marks it a failure; the quality plane marks it a success
D) Neither plane can detect it

---

### Question 6: The faithfulness signal

What does the **faithfulness** signal measure, and what input does it require?

A) Response latency; requires only the timestamp
B) Whether the output's claims are supported by the retrieved context; requires the output *and* the retrieved context
C) Whether the HTTP status was 2xx; requires only the status code
D) Tokens per second; requires the token counts

---

### Question 7: Drift

Why is drift described as a signal that "only exists over time" and is invisible at the single-trace level?

A) Drift is an APM metric that traces cannot store
B) Each individual run still passes; the degradation only appears when you aggregate a quality signal across a time window and look at the slope
C) Drift is the same as a single failed trace
D) Drift can only be measured inline during a run

---

### Question 8: Retries and trace identity

A transient error causes a worker's model call to be retried twice before succeeding. What is the correct tracing design?

A) Three separate traces, one per attempt
B) One trace; all three attempts are spans under the worker's `invoke_agent`, distinguished by a `retry.attempt` attribute
C) One span that overwrites itself on each retry, keeping only the success
D) Drop the failed attempts entirely so the trace looks clean

---

### Question 9: Sampling

Why is **tail sampling** the right default for agent traces over head sampling?

A) It is cheaper because it never buffers traces
B) It decides retention *after* the trace completes, when error status, quality score, and cost are known — so it keeps the rare failed/low-quality/high-cost runs
C) It samples a fixed 1% uniformly, guaranteeing fairness
D) It decides at span start, before the outcome is known

---

### Question 10: Platform selection

Which statement best reflects the Chapter 4 guidance for choosing an observability platform?

A) Pick whichever has the longest feature checklist
B) Derive weighted requirements from your own system and run a real proof-of-concept on your own traces; pick a vendor SDK lock-in only if needed
C) Always build your own on OTel Collector + ClickHouse — buying is never justified
D) Choose by the slickest vendor demo

---

## Answer Key

1. **C** — `invoke_agent` is the operation for one agent's invocation; `chat` is a model inference and `execute_tool` is a tool call. (Ch. 1)
2. **B** — Token usage attributes live on the `chat`/inference span, the unit that actually consumed tokens. (Ch. 1)
3. **B** — OTel context propagation: the `invoke_agent` span is the current span, so spans opened inside the loop nest under it without passing objects around. (Ch. 1)
4. **B** — `Resource` attributes are set once per process and stamped on every span; that is exactly what enables deploy-segmented quality comparison in Ch. 3. (Ch. 1, Ch. 3)
5. **B** — APM sees a fast `200` and calls it success; only the faithfulness signal catches the unsupported citation. This is the core agent-vs-APM blind spot. (Ch. 2)
6. **B** — Faithfulness is whether output claims are grounded in the retrieved context, so it requires both the output and the context to judge. (Ch. 2)
7. **B** — Every individual run passes; drift appears only as the slope of an aggregated quality signal over time, segmented by deploy. (Ch. 2)
8. **B** — One logical run is one trace; retries are sibling spans under it, distinguished by `retry.attempt` so they aren't confused with legitimate repeated calls. (Ch. 3)
9. **B** — Tail sampling decides after the trace completes, when outcome signals are known, so it retains the rare failures uniform/head sampling would discard. (Ch. 3)
10. **B** — Requirements-driven, weighted, and validated with a real POC on your own traces — not a checklist, not a demo, and build only behind a hard constraint. (Ch. 4)
