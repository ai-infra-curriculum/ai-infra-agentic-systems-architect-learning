# Chapter 4 — RAG and Knowledge Stores as Architecture

Long-term memory and the shared knowledge base from [Chapter 3](03-memory-tiers-and-state.md) have to be *retrieved* to be useful — pulled into the working window just-in-time, the right slice at the right moment. That retrieval layer is **RAG** (retrieval-augmented generation), and at the architect's altitude RAG is not "embed chunks, do cosine similarity, stuff the top-k into the prompt." That's the implementation. The architecture is the set of decisions *around* it: which store(s) hold the knowledge, how retrieval is structured, how fresh the index stays, and — the part teams most often skip — how you'll *know* the retrieval is good before it ever reaches the model.

This chapter treats the knowledge store as a component you select and the retrieval pipeline as a topology you design, both on stated trade-offs.

## Store selection: vector, graph, hybrid

The first decision is what kind of store backs retrieval. There is no universal answer; there's a fit between the store's strengths and the *shape of the questions* your agent asks.

### Vector store

Content is embedded into vectors; retrieval finds chunks whose embeddings are nearest the query embedding. Strengths: semantic similarity ("find passages *about* X" even without keyword overlap), simple to operate, scales to huge corpora with approximate-nearest-neighbor indexes. Weaknesses: it retrieves *similar text*, which is not the same as *relevant facts* — it has no notion of relationships, so "who reports to the person who approved budget Y?" is a question it answers only by luck. It also has no structure: it can't reliably do "the most recent" or multi-hop reasoning.

### Knowledge graph

Entities are nodes, relationships are edges; retrieval traverses the graph. Strengths: explicit, multi-hop relationships ("trace the dependency chain from service A to the failing database"), precise structured queries, and an auditable answer (you can show the path). Weaknesses: building and maintaining the graph is expensive — entity extraction and edge construction are themselves hard problems — and it's weak at fuzzy semantic matching over free text.

### Hybrid

In practice, serious systems combine them. The dominant pattern: a vector index for "find me the relevant region of knowledge by meaning" and a graph (or structured store) for "traverse the precise relationships within it." Hybrid also commonly means **vector + keyword (BM25)** retrieval fused together — semantic recall plus exact-term precision — because pure-vector retrieval famously misses exact identifiers, error codes, and proper nouns that keyword search nails.

```text
  question shape                       → store that fits
  ──────────────────────────────────── ──────────────────────────────
  "passages about <topic>"             → vector (semantic similarity)
  "exact term / ID / error code"       → keyword/BM25 (lexical)
  "relationship / multi-hop / path"    → graph (traversal)
  "most of the above, real corpus"     → HYBRID (vector+BM25, +graph
                                          when relationships matter)
```

### The selection matrix

The architect's artifact is a decision, defended:

| Need | Vector | Keyword/BM25 | Graph | Hybrid |
| --- | --- | --- | --- | --- |
| Semantic / fuzzy match | strong | weak | weak | strong |
| Exact term / identifier | weak | strong | medium | strong |
| Multi-hop relationships | weak | weak | strong | strong (with graph) |
| Auditable / explainable path | weak | medium | strong | medium–strong |
| Operational simplicity | strong | strong | weak | medium |
| Build/maintenance cost | low | low | high | medium–high |
| Freshness / easy updates | medium | strong | medium (re-extract) | medium |

The trap is reaching for a knowledge graph because it sounds sophisticated when 90% of your questions are semantic lookups a vector store answers more cheaply — or shipping pure-vector and watching it fail every time a user pastes an exact error code. Match the store to the question shape, and reach for hybrid when (as is common) the question shapes are mixed.

## Retrieval topology: the pipeline around the store

The store is one box. The retrieval *pipeline* is the architecture, and each stage is a decision point.

```text
  query ─▶ [rewrite] ─▶ [retrieve k] ─▶ [rerank] ─▶ [assemble] ─▶ window
             ▲              ▲ │            ▲            ▲
        expand/decompose    │ │ over-fetch  cross-encoder  budget-aware:
        the agent's vague   │ └─ (k large)  precision      fit top-n into
        query into good     │                              the JIT slice
        retrieval queries   └─ vector ⊕ BM25 fused (hybrid)  (Chapter 1)
```

- **Query rewriting / decomposition.** An agent's raw query is often a poor retrieval query. Rewriting (expand, add context from the conversation) and decomposition (split a multi-part question into separate retrievals) materially improve recall. For agents specifically, this is where **agentic retrieval** lives: the agent itself decides what to search for, iterates, and retrieves again if the first pass was thin — JIT retrieval ([Chapter 1](01-context-engineering-strategies.md)) as a loop, not a single shot.
- **Retrieve.** Over-fetch deliberately (large `k`) when a reranker follows; under-fetch when you don't, to protect the budget. Hybrid fusion (e.g., reciprocal-rank fusion of vector and BM25 results) happens here.
- **Rerank.** A cross-encoder (or LLM reranker) reorders the over-fetched candidates by true relevance to the query. This is the single highest-leverage precision stage: cheap to add, large effect, and it's what lets you over-fetch for recall and then narrow for precision. Architecturally, the reranker is where recall and budget reconcile.
- **Assemble.** Fit the final top-n into the window's retrieved-content slice from the [Chapter 1 budget](01-context-engineering-strategies.md). Assembly is budget-aware: deduplicate, drop near-identical chunks, and respect recency placement (Chapter 2) so the best evidence lands where attention is strongest.

The architect specifies this topology — which stages exist, where over-fetch happens, where the budget cap binds — not the embedding model's hyperparameters.

## Freshness: the index is a stale snapshot

A retrieval index is a *copy* of source knowledge, and copies drift. Freshness is a first-class architectural property, not an afterthought:

- **Update strategy.** Batch re-index (simple, but stale between runs) vs. incremental/streaming updates (fresh, more moving parts) vs. event-driven (re-embed on source change). Choose by how fast the truth changes and how wrong a stale answer is.
- **Invalidation.** When a source document is deleted or corrected, the index must reflect it — including in the per-user memory tiers, where a stale fact is also a *privacy* problem (the forgetting path from Chapter 3 has to reach the index).
- **The JIT alternative.** Sometimes the freshest design is to *not* pre-index at all and instead have the agent retrieve from the live source on demand (Chapter 1's JIT retrieval). An index trades freshness for speed; for fast-changing, small-surface data, live retrieval can be the better architecture.

## Retrieval evaluation: prove it before the model sees it

Here is the discipline that separates an architected RAG system from a hopeful one: **you evaluate retrieval as its own stage, independent of the final generation.** If the retriever hands the model the wrong context, no amount of prompt quality saves the answer — and a generation-only eval can't tell you *why* it failed. Decouple them.

Two families of retrieval metrics, both reference-based against a labeled set of (query → relevant-document) pairs:

- **Recall@k** — of the documents that *are* relevant, how many appeared in the top k? This is the recall question: did the right evidence make it into the candidate set at all? If recall@k is low, fix retrieval (better embeddings, hybrid, query rewriting); a reranker can't promote a document that was never fetched.
- **Precision@k / context precision** — of the k documents retrieved, how many are actually relevant? Low precision means you're spending budget on noise and inviting rot. This is where reranking earns its place.

Beyond ranking metrics, **RAG-specific quality checks** (the family popularized by frameworks like Ragas) score the assembled context and answer together: *context relevance* (is the retrieved context on-topic for the question?), *faithfulness / groundedness* (is the answer actually supported by the retrieved context, or hallucinated?), and *answer relevance* (does the answer address the question?). Faithfulness is the one that catches the dangerous failure: a fluent answer not grounded in the evidence.

```text
  RETRIEVAL EVAL HARNESS (runs in CI, like any eval — mod-304)
  ┌───────────────────────────────────────────────────────────┐
  │ labeled set: (query → relevant docs), + golden answers     │
  ├───────────────────────────────────────────────────────────┤
  │ retrieval stage:  recall@k ↑   precision@k ↑               │
  │ assembly stage:   context relevance ↑                      │
  │ generation stage: faithfulness ↑  answer relevance ↑       │
  ├───────────────────────────────────────────────────────────┤
  │ gate: regressions in recall@k or faithfulness BLOCK deploy │
  └───────────────────────────────────────────────────────────┘
```

This harness is the bridge to [mod-304](../mod-304-evaluation-harnesses/README.md): retrieval eval is just eval, gated in CI, with every production retrieval miss turned into a permanent test case. An architect who specifies the store and topology but not this harness has specified a system nobody can safely change.

## Starter guidance — the RAG architecture decision matrix

The artifact for [exercise-04](exercises/exercise-04-rag-architecture-for-agents.md). Fill it per agent.

```text
RAG ARCHITECTURE DECISION
Agent: ____________________   Corpus: ___________   Update rate: __________

1. QUESTION SHAPES (what does the agent actually ask?)
   □ semantic lookup   □ exact-term/ID   □ multi-hop/relationship   □ mixed

2. STORE CHOICE (justify against the matrix above)
   Primary store: vector / keyword / graph / hybrid  → because ______________
   Hybrid fusion?  vector ⊕ BM25 ? ___   + graph for relationships? ___

3. RETRIEVAL TOPOLOGY (which stages, and the budget cap)
   query rewrite/decompose: ___   over-fetch k = ___   rerank: ___
   final n into window = ___ tokens (must fit Chapter 1 retrieved slice)
   agentic (iterate) or single-shot? ___

4. FRESHNESS
   index update: batch / incremental / event-driven / live-JIT  → because ___
   invalidation path (incl. per-user forgetting): __________________________

5. RETRIEVAL EVAL (gate before the model)
   labeled set source: ____________   metrics gated in CI:
   □ recall@k ≥ ___   □ precision@k ≥ ___   □ faithfulness ≥ ___
   regression rule: every production miss → permanent test case
```

A RAG design you can defend on these five axes — question shape → store → topology → freshness → eval — is an architecture. The rest is implementation the engineering team can fill in.

## Key takeaways

- Choose the **store by question shape**: vector for semantic, keyword/BM25 for exact terms, graph for multi-hop relationships, **hybrid** (commonly vector+BM25, plus graph when relationships matter) for real mixed workloads — defend the choice against a matrix, don't reach for a graph because it sounds advanced.
- The architecture is the **retrieval topology**, not the store: query rewrite/decompose → over-fetch → **rerank** (the precision lever) → budget-aware assembly; agents make this a JIT loop, not a single shot.
- **Freshness is first-class**: pick an index-update strategy by how fast truth changes and how costly staleness is, build an invalidation path (which doubles as the per-user forgetting path), and consider live-JIT retrieval when freshness beats speed.
- **Evaluate retrieval as its own stage** before generation: recall@k and precision@k for ranking, plus faithfulness/groundedness and context relevance for quality — gated in CI, every production miss becoming a permanent case ([mod-304](../mod-304-evaluation-harnesses/README.md)).
