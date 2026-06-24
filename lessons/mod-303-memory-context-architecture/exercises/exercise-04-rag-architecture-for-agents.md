# exercise-04: RAG Architecture for Agents

**Estimated effort:** 3 hours

## Objective

Design the RAG architecture for an agent on stated trade-offs: choose the store(s), draw the retrieval topology, set a freshness strategy, and — the part that makes it an *architecture* — specify the **retrieval-eval harness** that proves retrieval is good before it reaches the model. You'll produce a filled decision matrix and a small working slice that demonstrates retrieval eval (recall@k and faithfulness) as a gate.

## Background

This exercise covers material from:

- [Chapter 4 — RAG and Knowledge Stores as Architecture](../04-rag-and-knowledge-stores.md)
- [Chapter 1 — Context Engineering as a Token Budget](../01-context-engineering-strategies.md) (retrieval fits a budgeted slice)
- [Chapter 3 — Where State Lives](../03-memory-tiers-and-state.md) (the shared knowledge base is a long-term tier)

Use any embedding model and any vector store (a local one like a FAISS/Chroma-style index, or even an in-memory cosine search, is fine). A keyword index (BM25) is needed for the hybrid task.

## Prerequisites

- A small corpus (50–500 documents) the agent should reason over. Mixed content is best: prose *and* exact identifiers/codes.
- An embedding model and a vector index.
- A labeled set of ~15–25 `(query → relevant doc id[s])` pairs and a handful of golden answers (you write these — they are the eval ground truth).

## Tasks

### 1. Characterize the question shapes

Inspect your corpus and intended queries. Classify the question shapes: semantic lookup, exact-term/ID, multi-hop/relationship, or mixed. This classification drives the store choice — do it first, with evidence (sample queries), not by preference.

### 2. Choose the store(s) against the matrix

Using the [Chapter 4 selection matrix](../04-rag-and-knowledge-stores.md#the-selection-matrix), choose your store: vector, keyword, graph, or hybrid. **Justify it against the matrix** — name the needs that drove the choice. If your question shapes are mixed (they usually are), implement **hybrid** retrieval: fuse vector and BM25 results (reciprocal-rank fusion is a fine default).

### 3. Build the retrieval topology

Implement the pipeline, not just the store:

- **Over-fetch** a large `k` from the store(s).
- **Rerank** the candidates (a cross-encoder, or an LLM-as-reranker prompt) down to a final `n`.
- **Assemble** the final `n` into a fixed token budget (the [Chapter 1](../01-context-engineering-strategies.md) retrieved-content slice) — deduplicate and respect recency placement.

Optionally add **query rewriting** (expand/decompose the agent's raw query before retrieval) and note the effect.

### 4. Set the freshness strategy

State, in writing: how the index updates (batch / incremental / event-driven / live-JIT) and *why*, given how fast your corpus changes and how costly a stale answer is. Describe the **invalidation path** for a deleted/corrected document — and connect it to the per-user **forgetting path** from [Chapter 3](../03-memory-tiers-and-state.md) if any retrieved data is per-user.

### 5. Build the retrieval-eval gate

This is the graded core. Using your labeled set:

- Compute **recall@k** and **precision@k** for the retrieval stage.
- Compute **faithfulness** (is the generated answer supported by the retrieved context?) and **answer relevance** for the generation stage — an LLM judge with a 1–3 rubric is acceptable.
- Define a **gate**: a threshold on recall@k and faithfulness that a change must clear. Show one change (e.g., adding the reranker, or switching vector→hybrid) that moves a metric, and show the gate catching a regression you induce on purpose.

## Starter guidance

```python
# retrieval topology + eval — skeleton
def retrieve(query: str, k_over: int = 30) -> list[str]:
    """Over-fetch: fuse vector + BM25 (hybrid). Return candidate doc ids."""
    raise NotImplementedError

def rerank(query: str, candidates: list[str], n: int = 5) -> list[str]:
    """Cross-encoder or LLM reranker. Return top-n by true relevance."""
    raise NotImplementedError

def assemble(doc_ids: list[str], token_budget: int) -> str:
    """Dedup, recency-place, fit into the Chapter 1 retrieved slice."""
    raise NotImplementedError

# --- retrieval eval ---
def recall_at_k(retrieved: list[str], relevant: set[str], k: int) -> float:
    top = set(retrieved[:k])
    return len(top & relevant) / max(1, len(relevant))

def precision_at_k(retrieved: list[str], relevant: set[str], k: int) -> float:
    top = retrieved[:k]
    return sum(1 for d in top if d in relevant) / max(1, k)

# faithfulness(answer, context) -> 1..3  via an LLM judge (groundedness rubric)

GATE = {"recall_at_5": 0.80, "faithfulness": 2.5}  # tune to your corpus
def passes_gate(metrics: dict) -> bool:
    return all(metrics[m] >= thr for m, thr in GATE.items())
```

```text
RAG ARCHITECTURE DECISION (fill and submit)
  Question shapes: ____________________________________
  Store: vector / keyword / graph / HYBRID  → because __________________
  Topology: rewrite? __  over-fetch k=__  rerank→n=__  budget=__ tokens
  Freshness: batch / incremental / event / live-JIT  → because __________
  Eval gate: recall@5 ≥ ____  faithfulness ≥ ____   regression rule: ____
```

## Acceptance criteria

You can demonstrate that:

- You classified **question shapes** from real sample queries and chose a store **justified against the matrix**.
- You implemented a **topology**, not just a store: over-fetch → rerank → budget-aware assembly (and hybrid fusion if your shapes are mixed).
- You stated a **freshness strategy** with an invalidation path tied to the corpus's change rate.
- You compute **recall@k, precision@k, faithfulness, and answer relevance** on a labeled set.
- You have a **gate** with thresholds, and you showed it **catch an induced regression**.

## Reflection

In `NOTES.md`:

1. Which question shape was your pure-vector retrieval worst at, and how much did hybrid (or reranking) improve recall@k for it? Give the numbers.
2. Where did faithfulness diverge from answer-relevance — i.e., a fluent answer that wasn't grounded? What in the topology let that through?
3. How did the retrieved-content token budget (Chapter 1) constrain your final `n`, and what did you trade off to fit it?

## Stretch goals

- Add a **graph** layer for one multi-hop question your vector/BM25 retrieval can't answer, and show the traversal producing the right evidence path.
- Make retrieval **agentic**: let the agent re-query when first-pass recall is thin (a JIT loop), and measure the recall lift vs. the extra latency/tokens.
- Wire the eval gate into a **CI check** ([mod-304](../../mod-304-evaluation-harnesses/README.md)) and turn one production-style retrieval miss into a permanent test case.
