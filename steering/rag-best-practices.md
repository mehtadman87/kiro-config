---
title: RAG Optimization Best Practices
inclusion: manual
---

# RAG Optimization Best Practices

Research-validated techniques for building production-ready Retrieval-Augmented Generation systems. Covers chunking, hybrid search, reranking, and pipeline optimization (2025-2026).

---

## Chunking Strategies

**Semantic chunking (recommended):** Split documents by paragraph, section, or logical unit rather than fixed token counts. Semantic boundaries preserve meaning and improve retrieval relevance.

**Chunk size guidelines:**
- 256-512 tokens: best for precise, fact-based retrieval (Q&A, lookup)
- 512-1024 tokens: balanced for most use cases
- 1024-2048 tokens: better for tasks requiring broader context (summarization, analysis)
- Include 10-20% overlap between chunks to preserve context at boundaries

**Metadata enrichment:** Attach metadata to each chunk: source document, section title, page number, creation date, document type. This enables filtered retrieval and improves relevance.

---

## Retrieval Strategies

**Hybrid search (recommended for production):**
- Run sparse retrieval (BM25 keyword matching) and dense retrieval (vector similarity) in parallel
- Merge results using reciprocal rank fusion or weighted combination
- Recommended weighting: 70% vector similarity, 30% BM25 keyword matching
- Hybrid search consistently outperforms either approach alone, especially for queries mixing technical terms with natural language

Source: "Building Production-Ready RAG Systems" (2025) [getathenic.com/blog/production-rag-systems-complete-guide]

**Reranking:** After initial retrieval, apply a cross-encoder reranker to re-score and reorder results. This is the single highest-impact optimization for RAG quality. Reranking improves the ranking of retrieved results; hybrid search improves what gets retrieved in the first place. Both are needed.

**Query transformation:**
- Query expansion: rephrase the user query into multiple variants to improve recall
- Hypothetical document embedding (HyDE): generate a hypothetical answer, embed it, and use that embedding for retrieval
- Step-back prompting: ask a more general question first to retrieve broader context, then narrow down

---

## RAG Pipeline Optimization

**Adaptive RAG:** Dynamically adjust retrieval strategy based on query complexity. Simple factual queries may need only 1-2 chunks; complex analytical queries may need 5-10 chunks from multiple sources. Use a lightweight classifier to route queries to the appropriate retrieval depth.

Source: "Dynamic Retrieval-Augmented Generation" (2026) [emergentmind.com/topics/adaptive-rag]

**Context compression:** After retrieval, compress chunks before injecting into the LLM context. Remove irrelevant sentences, deduplicate overlapping content, and extract only the passages most relevant to the query. This reduces token usage and improves answer quality.

**Citation and grounding:** Require the LLM to cite specific chunk IDs or source passages when generating answers. This enables verification and builds user trust. Implement automated citation checking: verify that cited passages actually support the claims made.

---

## RAG Anti-Patterns

- Retrieving too many chunks (more than 10) dilutes relevance and wastes tokens
- Using fixed chunk sizes without considering document structure
- Skipping reranking (the single most impactful optimization most teams skip)
- Not evaluating retrieval quality independently from generation quality
- Treating RAG as a one-time setup rather than a continuously tuned pipeline
