---
name: rag-embeddings-and-vector-stores
description: Choosing embedding models (MTEB, BGE) and vector stores (pgvector, Qdrant, plus Chroma/FAISS for prototypes), index types (HNSW vs IVFFlat), metadata filtering and normalization for RAG. Use this skill whenever the user picks an embedding model, a vector database, sets up pgvector or Qdrant, or asks about similarity metrics and indexes.
level: 2-core
---

# 04 - Embeddings and Vector Stores

**Simple-ah solna:** Embedding model = text-ah numbers-ah maathura machine. Vector store = andha numbers-ah store panni "idhukku nearest edhu" nu seekiram kandupidikkura DB.

## Embedding model
- Use the **MTEB leaderboard** to shortlist, then **test on your own golden set**. Leaderboard scores are on public data, not yours.
- Good open baseline: `BAAI/bge-base-en-v1.5` (768 dims). For multilingual data check BGE-M3 and similar.
- BGE models expect a **query instruction** for retrieval: `"Represent this sentence for searching relevant passages: "` + query. Documents are embedded without it.
- Normalize vectors (`normalize_embeddings=True`) so inner product = cosine.
- **Never mix** embedding models in one index. Changing the model = re-embed everything.

## Vector store choice
| Option | Pick when | Notes |
|---|---|---|
| **pgvector** (Postgres) | You already use Postgres/Django; want ACID, joins, filters in one DB | Exact search by default (perfect recall); add HNSW or IVFFlat for speed; `vector` type indexes up to 2,000 dims |
| **Qdrant** | You want a dedicated engine with built-in hybrid (dense + sparse) and rich filtering | Query API with `prefetch` + RRF fusion |
| Chroma / FAISS | Local prototypes, notebooks | Search by name; fine for learning, plan a move for production |

## pgvector quickstart
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE chunks (
  id bigserial PRIMARY KEY,
  doc_id text, source text, page int, text text,
  embedding vector(768)
);
-- approximate index (HNSW: better speed/recall tradeoff, slower build, more memory)
CREATE INDEX ON chunks USING hnsw (embedding vector_cosine_ops);
-- query: <=> is cosine distance, so similarity = 1 - distance
SELECT id, text, 1 - (embedding <=> %s) AS cosine_similarity
FROM chunks ORDER BY embedding <=> %s LIMIT 20;
```
- HNSW vs IVFFlat: HNSW has the better speed-recall tradeoff; IVFFlat builds faster and needs a training step (data must exist first).
- Raise `hnsw.ef_search` for better recall at the cost of speed.
- Filtered search: add a normal index on the filter column (e.g. tenant, doc type).
- From Python/Django use the `pgvector` Python package (see its README for Django support).

## Common mistakes
- Picking the DB first and never testing embedding models.
- Forgetting the query instruction for BGE-style models.
- Comparing un-normalized vectors with inner product.
- Approximate index and no recall check: always compare against exact search on a sample.

## Done when
- [ ] 2-3 embedding models compared on your golden set (hit-rate@k).
- [ ] Index + metadata filters working; recall vs exact search checked.
- [ ] Re-embedding procedure documented (model change, version tag).

## Learn more
- MTEB repo: https://github.com/embeddings-benchmark/mteb, leaderboard: https://huggingface.co/spaces/mteb/leaderboard, paper: https://arxiv.org/abs/2210.07316
- BGE / FlagEmbedding: https://github.com/FlagOpen/FlagEmbedding
- pgvector: https://github.com/pgvector/pgvector
- Qdrant hybrid queries: https://qdrant.tech/documentation/search/hybrid-queries/
