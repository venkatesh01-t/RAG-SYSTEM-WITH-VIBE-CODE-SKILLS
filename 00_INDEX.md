---
name: rag-skills-index
description: Master index and learning path for the end-to-end RAG skill set (13 skills, ordered low to high level), with usage instructions. Use this file first whenever the user asks which RAG skill to read, wants the roadmap, or is starting a RAG project.
---

# RAG Skills - Start to End (Low -> High)

Goal: build a RAG system whose hallucination is **as low as possible and measured**, and which says "I don't have enough information" when evidence is missing.
Honest note: 100% zero hallucination cannot be guaranteed by any RAG design. These skills give layered defenses plus measurement.

## Learning path
| Level | # | Skill file | You learn |
|---|---|---|---|
| 1 Foundation | 01 | 01-rag-fundamentals.md | Pipelines, embeddings, why hallucination happens |
| 1 Foundation | 02 | 02-ingestion-and-parsing.md | Clean text + metadata (Docling) |
| 1 Foundation | 03 | 03-chunking.md | Chunking strategies and tuning |
| 2 Core | 04 | 04-embeddings-and-vector-stores.md | Embedding models, pgvector, Qdrant |
| 2 Core | 05 | 05-hybrid-retrieval-and-reranking.md | BM25 + dense + RRF + reranker |
| 2 Core | 06 | 06-grounded-generation.md | Strict prompt, citations, JSON, abstain |
| 3 Reliability | 07 | 07-evaluation.md | Golden set, metrics, RAGAS/DeepEval |
| 3 Reliability | 08 | 08-anti-hallucination-guardrails.md | Gate, verifier, fail-closed layers |
| 4 Advanced | 09 | 09-contextual-retrieval-and-query-transforms.md | Contextual retrieval, query rewriting |
| 4 Advanced | 10 | 10-agentic-rag-and-self-correction.md | Agent loops, CRAG/Self-RAG ideas |
| 4 Advanced | 11 | 11-graphrag.md | Corpus-wide questions, knowledge graphs |
| 5 Production | 12 | 12-observability-security-deployment.md | Langfuse, access control, injection, deploy |
| 5 Production | 13 | 13-capstone-build-playbook.md | Full build plan + runnable v0 |

Also read: **ANALYSIS.md** (which steps matter most) and **SOURCES.md** (research results + every link).

## How to study
1. Read 01 -> 06, then immediately run the v0 code in 13. Do not wait to "finish theory".
2. Build the golden set (07) before tuning anything.
3. Improve one thing at a time; log numbers.
4. Touch levels 4-5 only when your metrics say you need them.

## Using these as Claude skills
Each file starts with `name` + `description` frontmatter. To install as skills (Claude Code / Claude.ai skills format), place each file as `<skill-name>/SKILL.md`. The zip includes a ready `skills/` folder with that layout.
Tip: after installing, ask Claude things like "use rag-evaluation to build my golden set" or "use rag-capstone-build-playbook to scaffold the repo".

## Limits of this pack
- Links were surfaced by web search on 2026-11-21; open each once to confirm it still loads.
- Code is a teaching skeleton: pin library versions and check current docs (RAGAS, DeepEval, LangChain and model names change often).
- Numbers quoted from papers/blogs are from their own test sets, not yours.
