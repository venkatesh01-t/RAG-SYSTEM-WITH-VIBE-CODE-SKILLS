---
name: rag-capstone-build-playbook
description: End-to-end playbook to build a low-hallucination RAG system from scratch in Python (and later Django + pgvector) - milestones with acceptance criteria and a complete minimal reference implementation (parse, chunk, hybrid retrieve, rerank, score gate, grounded generation, citation check, abstain). Use this skill whenever the user says "build a RAG system start to end", "RAG agent project plan", or wants one runnable skeleton that ties all other RAG skills together.
level: 5-production
---

# 13 - Capstone: Build It Start to End

**Simple-ah solna:** Munnaadi 12 skills-um oru oru piece. Idhu andha pieces-ah onnaa serkura plan. Mudhala v0 chinnadhaa build pannu, evaluate pannu, appuram ovvoru improvement-ah measure panni add pannu.

## Milestones (do them in this order)
| # | Milestone | Skills | Accept when |
|---|---|---|---|
| M0 | Pick ONE use case + 20-50 docs; write 50 golden questions (>=15% unanswerable) | 07 | Golden set committed |
| M1 | v0 pipeline below (in-memory) | 01-06 | End-to-end answer with citations |
| M2 | Evaluation harness + baseline numbers | 07 | One command prints all metrics |
| M3 | Tune chunking, embeddings, hybrid, rerank - one change at a time | 03-05 | Hit-rate and precision improved, logged |
| M4 | Gate + verifier + fail-closed abstention | 08 | Unanswerable set mostly abstains; false-abstain acceptable |
| M5 | Move to pgvector (or Qdrant) + metadata filters | 04 | Same metrics as in-memory |
| M6 | Django REST API + background ingestion | 12 | `/ask` works, ingestion idempotent |
| M7 | Tracing + feedback loop + security tests | 12 | Traces visible, access test passes |
| M8 | Only if metrics demand it: contextual retrieval, agentic loop, GraphRAG | 09-11 | Beats baseline on golden set |

Set your own numeric targets (for example faithfulness and correct-abstain thresholds) after you see baseline numbers.

## Suggested repo layout
```
rag/
  data/raw/            # source files
  data/golden.jsonl    # evaluation questions
  rag/ingest.py  rag/chunk.py  rag/index.py  rag/retrieve.py
  rag/generate.py  rag/guardrails.py  rag/pipeline.py
  eval/run_eval.py
  api/ (Django later)
```

## Reference implementation v0 (single file, no framework)
```python
# pip install docling langchain-text-splitters sentence-transformers rank-bm25 FlagEmbedding anthropic numpy
import re, numpy as np
from docling.document_converter import DocumentConverter
from langchain_text_splitters import RecursiveCharacterTextSplitter
from sentence_transformers import SentenceTransformer
from rank_bm25 import BM25Okapi
from FlagEmbedding import FlagReranker
# from skill 06: generate, citations_ok, ABSTAIN, client, MODEL
# from skill 05: rrf
# from skill 08: verify

embedder = SentenceTransformer("BAAI/bge-base-en-v1.5")
reranker = FlagReranker("BAAI/bge-reranker-v2-m3", use_fp16=True)
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=150)
QUERY_PREFIX = "Represent this sentence for searching relevant passages: "
MIN_SCORE = 0.3        # CALIBRATE on your golden set (skill 08)

tok = lambda s: re.findall(r"\w+", s.lower())

def ingest(paths):
    conv, chunks = DocumentConverter(), []
    for p in paths:
        md = conv.convert(p).document.export_to_markdown()
        for i, t in enumerate(splitter.split_text(md)):
            chunks.append({"id": f"{p}#{i}", "source": p, "text": t})
    return chunks

def build_index(chunks):
    emb = embedder.encode([c["text"] for c in chunks], normalize_embeddings=True)
    bm25 = BM25Okapi([tok(c["text"]) for c in chunks])
    return emb, bm25

def retrieve(q, chunks, emb, bm25, k=50):
    qv = embedder.encode([QUERY_PREFIX + q], normalize_embeddings=True)[0]
    dense = [int(i) for i in np.argsort(-(emb @ qv))[:k]]
    sparse = [int(i) for i in np.argsort(-bm25.get_scores(tok(q)))[:k]]
    return [chunks[i] for i in rrf([dense, sparse])[:k]]

def rerank(q, cands, top=6):
    scores = reranker.compute_score([[q, c["text"]] for c in cands], normalize=True)
    ranked = sorted(zip(cands, scores), key=lambda x: -x[1])[:top]
    return [{**c, "score": float(s)} for c, s in ranked]

def ask(q, chunks, emb, bm25):
    top = [c for c in rerank(q, retrieve(q, chunks, emb, bm25)) if c["score"] >= MIN_SCORE]
    if not top:
        return ABSTAIN                                   # L2 gate
    out = generate(q, top)                               # L3 prompt
    if out["status"] == "answered":
        if not citations_ok(out, len(top)):              # L4
            return ABSTAIN
        if not verify(out["answer"], top):               # L5
            return ABSTAIN
    return out
```
Run it on 3 documents first. Then run your golden set through `ask()` and score it (skill 07).

## Common mistakes
- Building the agent first and evaluation never.
- Changing 4 things at once, so you never learn what helped.
- No unanswerable questions in the golden set (then hallucination is invisible).
- Over-engineering with GraphRAG / multi-agent before a measured baseline.

## Done when
- [ ] M0-M4 complete with a results table in the repo.
- [ ] You can explain, for any failed question, which layer failed.
- [ ] Abstention works: unanswerable questions get "I don't have enough information".

## Learn more
- Runnable notebooks for each technique: https://github.com/NirDiamant/RAG_Techniques
- LangChain RAG agent tutorial: https://docs.langchain.com/oss/python/langchain/rag
- LlamaIndex starter (local models): https://developers.llamaindex.ai/python/framework/getting_started/starter_example_local/
- Local private RAG agent walkthrough: https://www.freecodecamp.org/news/build-a-private-rag-qa-ai-agent-for-your-documents-using-langchain/
