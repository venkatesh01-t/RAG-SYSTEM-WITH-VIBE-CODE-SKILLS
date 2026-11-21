# Analysis: Which Steps Matter Most

## Priority ranking (impact x effort), with reasoning
| Rank | Step | Why it ranks here | Skill |
|---|---|---|---|
| 1 | **Golden set + evaluation** | Without it every change is a guess. Includes unanswerable questions, the only way to see hallucination. | 07 |
| 2 | **Data quality (parsing, dedupe, freshness)** | Wrong or garbled source = wrong answer, however good the model. Faithful is not the same as true. | 02 |
| 3 | **Chunking + context per chunk** | Lost context makes the right chunk un-findable. Contextual retrieval cut top-20 retrieval failures by 35-49% in Anthropic's tests. | 03, 09 |
| 4 | **Hybrid retrieval + reranking** | Dense misses exact identifiers; reranking sharpens the shortlist. Adding reranking brought Anthropic's failure reduction to up to 67%. | 05 |
| 5 | **Abstain gate + strict prompt + citations** | Cheapest, highest-leverage hallucination control: never answer when evidence is weak. | 06, 08 |
| 6 | **Verifier (claim-level check)** | Catches subtle unsupported claims; costs an extra LLM call. | 08 |
| 7 | Observability + feedback loop | Turns production failures into better golden sets. | 12 |
| 8 | Agentic loops, GraphRAG | Real wins only for multi-hop or corpus-wide questions; heavy cost and complexity. | 10, 11 |

## What NOT to do early
- Agents, multi-agent, GraphRAG before a measured baseline.
- Fine-tuning the LLM to "fix hallucination" (retrieval and gating usually give more per effort).
- Choosing a vector DB by hype; pick by your stack (Postgres/Django -> pgvector) and test embeddings instead.

## Symptom -> likely layer -> fix
| Symptom | Likely cause | Try |
|---|---|---|
| Right doc exists, answer says "not found" | Retrieval miss | Chunking, hybrid, query transforms, contextual retrieval |
| Misses exact codes / names | Dense-only retrieval | Add BM25 + RRF |
| Answer mixes in unrelated facts | Noisy context | Rerank, fewer chunks, tighter gate |
| Confident answer with no source support | Prompt/gate too loose | Strict prompt, score gate, verifier |
| Fake or wrong citations | No programmatic check | Citation validation, verifier |
| Good retrieval, still wrong | Source is wrong/outdated | Fix data (02), versioning |
| Fails on "summarize all documents" | Top-k cannot cover the corpus | Consider GraphRAG (11) |
| Follow-up questions fail in chat | Missing conversation context | Query condensing (09) |

## Residual risk (be honest with users and stakeholders)
1. **Faithfulness != truth.** Grounded in a wrong document is still wrong.
2. **LLM judges are fallible.** Spot-check manually.
3. **Prompt injection via retrieved text** is real (skill 12).
4. **High-stakes domains** need human review and visible citations.
5. **Benchmarks are not your data.** MTEB scores, blog percentages and paper wins must be re-measured on your golden set.

## Evidence base used (all verified in this research pass)
- Anthropic Contextual Retrieval: 35% (contextual embeddings), 49% (+ contextual BM25), up to 67% (+ reranking) lower top-20 retrieval failure rate, on Anthropic's test sets.
- Qdrant: hybrid search adds cost; measure whether the gain justifies it.
- GraphRAG (Microsoft): wins on comprehensiveness and diversity (~70-80% in LLM-judged tests) for global questions; indexing is expensive, start small.
- CRAG: retrieval evaluator triggers Correct / Incorrect / Ambiguous actions, with web search fallback.
- RAGAS faithfulness = supported claims / total claims in the answer.
