---
name: rag-fundamentals
description: Core concepts of Retrieval-Augmented Generation (RAG) - what it is, the indexing and query pipelines, Naive vs Advanced vs Modular RAG, embeddings and cosine similarity, and why hallucination happens. Use this skill whenever the user is starting a RAG project, asks "what is RAG", wants to understand embeddings, vector search or the RAG pipeline, or needs the mental model before building anything.
level: 1-foundation
---

# 01 - RAG Fundamentals

**Simple-ah solna:** LLM-kitta direct-ah kelvi kettaa adhu memory-la irundhu guess pannum. RAG-la nee un documents-la irundhu sariyaana paragraphs-ah edutthu LLM-ku "idha vachu mattum answer sollu" nu kudukkura. Proof kuduthu answer vaanguraadhu dhaan RAG.

## 1. The two pipelines

```
INDEXING (offline, once + on updates)
  load -> parse -> clean -> chunk -> embed -> store (vector DB + keyword index + metadata)

QUERY (online, every question)
  question -> (rewrite) -> retrieve -> (rerank) -> build prompt -> generate -> (verify) -> answer + citations
```

Everything else in this skill set is an improvement to one box in this diagram.

## 2. Three generations (from the Gao et al. survey)
- **Naive RAG**: index -> retrieve -> read. Simple "retrieve-read".
- **Advanced RAG**: adds pre-retrieval (query rewriting, better indexing) and post-retrieval (reranking, compression).
- **Modular RAG**: components (search, memory, routing, verification) are swappable modules; agentic RAG lives here.

## 3. Embeddings and similarity in 10 lines
An embedding model turns text into a vector. Similar meaning -> vectors point in a similar direction. If vectors are normalized, dot product = cosine similarity.

```python
# pip install sentence-transformers
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("BAAI/bge-base-en-v1.5")
docs = ["Madurai is a city in Tamil Nadu.", "Python is a programming language."]
q = "Which state is Madurai in?"
d = model.encode(docs, normalize_embeddings=True)
qv = model.encode([q], normalize_embeddings=True)
print(qv @ d.T)   # higher = more similar
```
Run it. Change the query. Watch which doc wins. This is the whole idea of vector search.

## 4. Why hallucination happens (map each cause to a skill)
| Cause | Fix lives in |
|---|---|
| Right chunk never retrieved, LLM fills the gap by guessing | 03, 05, 09 |
| Chunk cut in the middle, context lost | 03, 09 |
| Noisy / irrelevant chunks in prompt | 05 (rerank), 08 (gate) |
| Prompt lets the model use outside knowledge | 06 |
| Source document itself is wrong or outdated | 02, 12 |
| No way to detect a bad answer | 07, 08 |

## 5. Honest limit
RAG **reduces** hallucination a lot; it does **not** make it zero. A faithful answer built on a wrong document is still wrong. Goal = "very low, measured, and abstains when unsure".

## Common mistakes
- Jumping to LangChain/agents before understanding the plain pipeline.
- Judging quality by "it looked right on 3 questions".
- Treating the vector DB choice as the main decision (it is not; data quality and retrieval tuning matter more).

## Done when
- [ ] You can draw both pipelines from memory.
- [ ] You ran the cosine-similarity snippet and explained the result.
- [ ] You can name which box each hallucination cause belongs to.

## Learn more
- Original RAG paper (Lewis et al., 2020): https://arxiv.org/abs/2005.11401
- RAG survey (Gao et al.): https://arxiv.org/abs/2312.10997 and https://github.com/Tongji-KGLLM/RAG-Survey
- LlamaIndex "Introduction to RAG" (5 stages): https://developers.llamaindex.ai/python/framework/understanding/rag/
- LangChain RAG tutorial: https://docs.langchain.com/oss/python/langchain/rag
