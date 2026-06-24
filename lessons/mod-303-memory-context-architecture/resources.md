# Resources for mod-303-memory-context-architecture

Primary references for memory and context architecture. Verify against current docs — agent memory tooling and model context limits move fast.

## Context engineering and the window

- **Anthropic — Effective context engineering for AI agents** ([anthropic.com/engineering/effective-context-engineering-for-ai-agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)) — the canonical treatment of context as a finite resource: compaction, structured note-taking, sub-agent isolation, and just-in-time retrieval. Start here.
- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the orchestration patterns; sub-agent isolation as a context lever sits inside the orchestrator-workers pattern.
- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — real context-economics of isolated sub-agents returning distilled results.

## Context rot and long-context behavior

- **Chroma — Context Rot: How increasing input tokens impacts LLM performance** ([research.trychroma.com/context-rot](https://research.trychroma.com/context-rot)) — empirical study of quality degrading as the window grows; the basis for the "more context is not more capability" framing.
- **Liu et al. — Lost in the Middle: How Language Models Use Long Contexts** ([arxiv.org/abs/2307.03172](https://arxiv.org/abs/2307.03172)) — the foundational result on positional attention bias in long contexts.
- **Needle in a Haystack** ([github.com/gkamradt/LLMTest_NeedleInAHaystack](https://github.com/gkamradt/LLMTest_NeedleInAHaystack)) — the canonical retrieval-depth probe; extend it with distractors as in [Chapter 2](02-context-rot.md).

## Memory tiers and agent state

- **LangGraph — Memory** ([langchain-ai.github.io/langgraph/concepts/memory/](https://langchain-ai.github.io/langgraph/concepts/memory/)) — short-term (thread) vs. long-term (cross-thread) memory and how persistence/checkpointing draws the boundary.
- **MemGPT / Letta — Towards LLMs as Operating Systems** ([arxiv.org/abs/2310.08560](https://arxiv.org/abs/2310.08560)) — tiered memory (in-context vs. external) with explicit paging between them; the OS analogy for memory hierarchy.
- **Mem0** ([github.com/mem0ai/mem0](https://github.com/mem0ai/mem0)) — a production memory layer with scoping (user/session/agent) that maps directly to the episodic/long-term tiers and scope boundaries in [Chapter 3](03-memory-tiers-and-state.md).

## RAG, vector, and knowledge-graph stores

- **Anthropic — Contextual Retrieval** ([anthropic.com/news/contextual-retrieval](https://www.anthropic.com/news/contextual-retrieval)) — improving chunk retrieval with context and hybrid (embedding + BM25) retrieval plus reranking.
- **Microsoft Research — GraphRAG** ([microsoft.github.io/graphrag/](https://microsoft.github.io/graphrag/)) — knowledge-graph-backed retrieval for multi-hop, relationship, and global questions vector search handles poorly.
- **pgvector** ([github.com/pgvector/pgvector](https://github.com/pgvector/pgvector)) and **FAISS** ([github.com/facebookresearch/faiss](https://github.com/facebookresearch/faiss)) — representative vector stores/indexes for the exercise-04 build.
- **Reciprocal Rank Fusion** (Cormack et al., [plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf)) — the simple, robust fusion method for combining vector and keyword result lists in hybrid retrieval.

## Retrieval evaluation

- **Ragas** ([docs.ragas.io](https://docs.ragas.io)) — RAG-specific metrics: faithfulness/groundedness, context relevance/precision, and answer relevance — the retrieval-eval harness in [Chapter 4](04-rag-and-knowledge-stores.md).
- **TREC and IR metrics** — recall@k and precision@k are standard information-retrieval measures; any IR reference (e.g., Manning, Raghavan & Schütze, *Introduction to Information Retrieval*) grounds the ranking-metric vocabulary.

> You decide *whether and how* memory and retrieval belong in the architecture here; the engineer-track `RAG & Memory` module covers *building* them. When you adopt a memory framework, you'll recognize exactly which tier and boundary it's implementing — and which it leaves to you.
