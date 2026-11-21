---
name: rag-hybrid-retrieval-and-reranking
description: Hybrid retrieval (dense vectors + BM25 keyword search fused with Reciprocal Rank Fusion) and cross-encoder reranking (bge-reranker) to raise retrieval accuracy in RAG. Use this skill whenever retrieval misses exact names, IDs or codes, when top-k results are noisy, or when the user asks how to improve retrieval quality beyond plain vector search.
level: 2-core
---

# 05 - Hybrid Retrieval and Reranking

**Simple-ah solna:** Vector search meaning-ah pidikkum, aana exact ID / product code-ah miss pannum. Keyword search (BM25) exact words-ah pidikkum. Rendaiyum serthu, appuram reranker-aal top-ah re-sort pannina retrieval romba strong.

## The retrieval funnel
```
dense top 50  \
               >-- RRF fusion --> top 50 --> reranker --> top 5-8 --> LLM
BM25 top 50   /
```

## Why hybrid
Dense retrieval can return the right topic yet miss an exact identifier copied into the query; sparse retrieval adds exact term matching. Hybrid merges both result lists. It costs extra storage, indexing and query work, so **measure** the gain on your golden set instead of assuming it.

## Reciprocal Rank Fusion (RRF)
```python
def rrf(rank_lists, k=60):
    """rank_lists: list of lists of ids, best first."""
    scores = {}
    for ranks in rank_lists:
        for r, doc_id in enumerate(ranks, start=1):
            scores[doc_id] = scores.get(doc_id, 0.0) + 1.0 / (k + r)
    return sorted(scores, key=scores.get, reverse=True)
```
No score normalization needed; it only uses ranks.

## BM25 locally
```python
# pip install rank-bm25
import re
from rank_bm25 import BM25Okapi
tok = lambda s: re.findall(r"\w+", s.lower())
bm25 = BM25Okapi([tok(c["text"]) for c in chunks])
sparse_ids = bm25.get_scores(tok(query)).argsort()[::-1][:50]
```
In production use the engine's own: Qdrant Query API (`prefetch` dense + sparse, then `rrf`), or Postgres full-text search alongside pgvector.

## Reranking with a cross-encoder
A reranker reads (query, passage) together and outputs a relevance score. It is slower than vector search, so run it only on the shortlist.
```python
# pip install -U FlagEmbedding
from FlagEmbedding import FlagReranker
reranker = FlagReranker("BAAI/bge-reranker-v2-m3", use_fp16=True)
pairs = [[query, c["text"]] for c in candidates]
scores = reranker.compute_score(pairs, normalize=True)   # 0..1 via sigmoid
ranked = sorted(zip(candidates, scores), key=lambda x: -x[1])[:6]
```
Keep the reranker score: skill 08 uses it as an **abstain gate**.

## Evidence
Anthropic reports that adding reranking on top of contextual embeddings + contextual BM25 cut the top-20 retrieval failure rate by up to 67% (on their test sets). Treat numbers as direction, not a promise for your data.

## Common mistakes
- Reranking only 5 candidates (nothing to reorder). Retrieve wide (50-100), rerank narrow.
- Sending 20 chunks to the LLM "to be safe": noise raises hallucination.
- Never testing dense-only vs hybrid on your queries.

## Done when
- [ ] Hit-rate@k and context precision compared: dense-only vs hybrid vs hybrid+rerank.
- [ ] Reranker scores logged per query.
- [ ] Final context = 4-8 chunks, not more.

## Learn more
- Qdrant hybrid search guide: https://qdrant.tech/documentation/search-tuning/hybrid-search/
- Qdrant hybrid queries (prefetch + RRF): https://qdrant.tech/documentation/search/hybrid-queries/
- BGE rerankers: https://huggingface.co/BAAI/bge-reranker-v2-m3
- Anthropic Contextual Retrieval (BM25 + rerank results): https://www.anthropic.com/engineering/contextual-retrieval
