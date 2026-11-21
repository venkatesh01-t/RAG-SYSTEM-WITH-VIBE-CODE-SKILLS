---
name: rag-agentic-and-self-correcting
description: Agentic RAG - letting the LLM decide when to retrieve, grade retrieved documents, rewrite the query and retry, route between tools/sources, with bounded loops and fail-closed abstention (CRAG/Self-RAG ideas, LangGraph agentic RAG). Use this skill whenever the user wants a RAG "agent", multi-step retrieval, tool use beyond a single vector search, or self-correcting retrieval - but only after the baseline pipeline is evaluated.
level: 4-advanced
---

# 10 - Agentic RAG and Self-Correction

**Simple-ah solna:** Normal RAG = oru thadavai retrieve, oru thadavai answer. Agentic RAG = LLM yosikkum: "idhukku retrieve pannanuma? Kedachadhu sariyaa? Illana query-ah maathi thirumba try pannalaama?" Idhu power, aana cost + complexity kooda varum.

## Build order (do not skip)
Baseline RAG -> evaluation (07) -> guardrails (08) -> **then** agentic loops, if the metrics show multi-step failures.

## The core loop (framework-free)
```python
def answer(question, max_rounds=2):
    q = question
    for _ in range(max_rounds + 1):
        chunks = retrieve_and_rerank(q)                      # skills 04-05
        good = [c for c in chunks if grade(question, c) == "yes"]
        if len(good) >= 2:
            out = generate(question, good)                   # skill 06
            if citations_ok(out, len(good)) and verify(out["answer"], good):   # skill 08
                return out
        q = rewrite(question, previous_query=q)              # reformulate and retry
    return ABSTAIN                                           # bounded + fail closed
```
`grade()` = a small LLM call: "Is this chunk relevant to the question? yes/no" (the LangGraph agentic-RAG tutorial uses exactly this binary grader). `rewrite()` = "Rewrite the query to retrieve better evidence".

## Patterns
| Pattern | Idea |
|---|---|
| Retrieve-on-demand | Retrieval is a tool; the agent may skip it for chit-chat |
| Document grading (CRAG-style) | Grade retrieved docs; if poor, rewrite, search again, or use another source |
| Routing | Pick the right index/tool (docs vs SQL vs web) per question |
| Decomposition | Split into sub-questions, answer each with citations, merge |
| Self-RAG | Model trained with reflection tokens to retrieve + critique (needs special models) |

## Design rules
1. **Bound the loop** (max rounds, max tokens, timeout).
2. **Fail closed**: out of budget -> abstain, not "best guess".
3. Log every step (query, grade, decision) for debugging (skill 12).
4. Each extra LLM call multiplies cost and latency; measure the accuracy gain per call.
5. More tools = more attack surface (skill 12).

## Frameworks
LangGraph (graph nodes/edges, state) and LlamaIndex agents/workflows both support this. Learn the plain loop first so you understand what the framework hides.

## Done when
- [ ] Evaluation shows agentic mode beats the single-pass baseline on multi-hop questions.
- [ ] Loop is bounded and tested with unanswerable questions (must abstain).
- [ ] Per-query cost/latency measured.

## Learn more
- LangGraph agentic RAG tutorial: https://docs.langchain.com/oss/python/langgraph/agentic-rag
- LangChain RAG agent: https://docs.langchain.com/oss/python/langchain/rag
- LlamaIndex agents with RAG tool: https://developers.llamaindex.ai/python/framework/getting_started/starter_example/
- CRAG: https://arxiv.org/abs/2401.15884 | Self-RAG: https://arxiv.org/abs/2310.11511
