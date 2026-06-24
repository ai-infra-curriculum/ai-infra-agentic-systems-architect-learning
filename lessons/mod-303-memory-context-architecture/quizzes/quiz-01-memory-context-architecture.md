# Quiz 1 — Memory & Context Architecture

Knowledge check for [mod-303](../README.md). Answers are at the bottom; try each question before scrolling. Covers all four chapters.

## Questions

### 1. The window as budget

Why does an architect set a usable context budget (say ~65% of capacity) below the model's hard window limit?

- A. The provider bills per token only above 65%.
- B. Cost/latency scale with every surviving token, and quality degrades well before the hard cap — so you reserve headroom and conserve signal.
- C. The tokenizer is inaccurate near the limit.
- D. There is no reason; always fill the window for maximum knowledge.

### 2. Stable before volatile

Why order the system prompt and tool definitions at the **front** of the window and churning history behind them?

- A. Models ignore the end of the window entirely.
- B. So prompt caching can amortize the stable prefix across turns.
- C. It reduces the token count of the stable parts.
- D. The order has no effect on cost or behavior.

### 3. Sub-agent isolation as budget

In budget terms, what does running a sub-task in an isolated sub-agent buy the parent?

- A. Nothing — the parent still pays for the sub-agent's full transcript.
- B. It converts an unbounded, high-token exploration into a bounded, low-token distilled return the parent can afford.
- C. It removes the need for any retrieval.
- D. It makes the sub-task run faster than in the parent.

### 4. Context rot

What is "context rot"?

- A. Embeddings degrading on disk over time.
- B. Quality declining non-monotonically as the window grows longer and noisier — attention dilutes, stale content distracts, the agent self-conditions on its drift.
- C. The vector index returning deleted documents.
- D. The model forgetting its system prompt after one turn.

### 5. Lost in the middle

You bury a critical fact at ~50% depth in a 150K-token window and the agent misses it, but it answers when the same fact is near the end. This is best explained by:

- A. A tokenizer bug.
- B. The lost-in-the-middle effect — attention favors the edges, so mid-window facts are dimmed.
- C. The fact being semantically irrelevant.
- D. Insufficient model parameters.

### 6. The distractor probe

Why add plausible-but-wrong "distractor" facts to a needle-in-a-haystack test instead of running it clean?

- A. To make the test cheaper.
- B. Because real agent context is full of stale-but-related content; the distractor variant catches the agent that will cite a stale tool output in production.
- C. Distractors are required by the OTel spec.
- D. A clean test is impossible to construct.

### 7. The promotion boundary

An engineer proposes writing every conversation turn to a vector store "so the agent remembers everything." The architect's objection is:

- A. Vector stores can't hold conversation turns.
- B. Promote-everything builds noise the agent later retrieves and rots on; memory must be curated, not accumulated.
- C. It violates read-your-writes consistency.
- D. Episodic memory must be system-scoped.

### 8. The scope boundary

What makes per-user memory isolation a *security* boundary rather than a quality nicety, and how is it best enforced?

- A. It isn't security-relevant; any filter is fine.
- B. One user's memory surfacing in another's session is a cross-tenant leak; enforce it with a partition key in the storage schema, not a `WHERE` clause in the query.
- C. Enforce it by making retrieval queries more specific.
- D. Enforce it by encrypting the embeddings.

### 9. Consistency per tier

Which consistency property does **long-term/semantic** memory most need that working memory does not?

- A. Read-your-writes within a single turn.
- B. Durability and versioning, so a corrected/superseded fact can replace an old one without forcing the agent to reason over contradictions.
- C. Sub-second latency.
- D. None — long-term memory needs the least of all tiers.

### 10. The forgetting path

Why must a memory architecture design a "forgetting path" from day one?

- A. To reduce storage cost only.
- B. Because "delete user U" must reach every per-user partition across episodic, long-term, and any retrieval index — designing the scope key in from the start makes it one operation, not an archaeology project.
- C. Because models forget facts anyway.
- D. It's optional; deletion can always be added later for free.

### 11. Store selection

Your agent mostly answers "trace the dependency chain from service A to the failing database" — multi-hop relationship questions. Which store fits best, and why?

- A. Pure vector — semantic similarity covers everything.
- B. A graph (or hybrid with graph) — explicit, traversable, auditable multi-hop relationships are exactly its strength and vector's weakness.
- C. Keyword/BM25 — exact terms only.
- D. Any store; the question shape doesn't affect the choice.

### 12. Pure-vector's blind spot

Why do serious RAG systems often fuse vector retrieval with keyword/BM25?

- A. BM25 is faster than vector search in all cases.
- B. Pure-vector retrieval misses exact identifiers, error codes, and proper nouns that lexical search nails; fusion gives semantic recall *and* exact-term precision.
- C. Vector search can't scale past 1,000 documents.
- D. BM25 eliminates the need for a reranker.

### 13. The reranker

In the retrieval topology, what role does the reranker play and why is it high-leverage?

- A. It generates the final answer.
- B. It reorders an over-fetched candidate set by true relevance — cheap to add, large precision gain, and it's what lets you over-fetch for recall then narrow for budget.
- C. It embeds the query.
- D. It invalidates stale index entries.

### 14. Retrieval eval, decoupled

Why evaluate retrieval (recall@k, precision@k) as its own stage, separate from the final generation?

- A. To save tokens on the judge.
- B. If the retriever hands the model the wrong context, no prompt saves the answer — and a generation-only eval can't tell you the failure was in retrieval. Decoupling localizes the fault.
- C. Generation eval is impossible for RAG.
- D. Recall@k and faithfulness measure the same thing.

### 15. Faithfulness

A RAG answer is fluent and well-written but asserts a claim not present in the retrieved context. Which metric is designed to catch this?

- A. Answer relevance.
- B. Faithfulness / groundedness — does the retrieved context actually support the answer?
- C. Recall@k.
- D. Precision@k.

## Answer key

1. **B** — Tokens cost on every surviving turn and quality degrades before the cap; reserve headroom and conserve signal ([Chapter 1](../01-context-engineering-strategies.md)).
2. **B** — A stable front lets prompt caching amortize it; volatile content goes behind ([Chapter 1](../01-context-engineering-strategies.md)).
3. **B** — Isolation spends a fresh budget in the child and returns a bounded distilled summary the parent can afford ([Chapter 1](../01-context-engineering-strategies.md)).
4. **B** — Non-monotonic quality decline from dilution, distraction, and self-conditioning as the window grows ([Chapter 2](../02-context-rot.md)).
5. **B** — Lost-in-the-middle: attention favors the edges, dimming mid-window facts ([Chapter 2](../02-context-rot.md)).
6. **B** — Distractors mimic real stale-but-related context and expose the agent that will cite stale output in production ([Chapter 2](../02-context-rot.md)).
7. **B** — Promote-everything is noise the agent rots on; memory must be curated ([Chapter 3](../03-memory-tiers-and-state.md)).
8. **B** — Cross-user surfacing is a tenant leak; partition by a schema key, not a query filter ([Chapter 3](../03-memory-tiers-and-state.md)).
9. **B** — Long-term memory needs durability and versioning so corrected facts replace old ones ([Chapter 3](../03-memory-tiers-and-state.md)).
10. **B** — Deletion must reach every per-user partition and index; the scope key designed in from the start makes it one operation ([Chapter 3](../03-memory-tiers-and-state.md)).
11. **B** — Multi-hop relationship questions are a graph's strength and a vector store's weakness ([Chapter 4](../04-rag-and-knowledge-stores.md)).
12. **B** — Pure-vector misses exact identifiers; vector+BM25 fusion gives semantic recall plus lexical precision ([Chapter 4](../04-rag-and-knowledge-stores.md)).
13. **B** — The reranker reorders over-fetched candidates by true relevance: cheap, high precision, reconciles recall with budget ([Chapter 4](../04-rag-and-knowledge-stores.md)).
14. **B** — Decoupling localizes the fault; a bad retriever can't be fixed by a good prompt, and generation-only eval hides it ([Chapter 4](../04-rag-and-knowledge-stores.md)).
15. **B** — Faithfulness/groundedness checks whether the retrieved context actually supports the answer ([Chapter 4](../04-rag-and-knowledge-stores.md)).
