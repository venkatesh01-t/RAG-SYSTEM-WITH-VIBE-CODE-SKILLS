---
name: rag-graphrag
description: When and how to use GraphRAG (knowledge-graph based RAG with community summaries) for corpus-wide "global" questions and relationship-heavy data, including costs and alternatives. Use this skill whenever the user asks about GraphRAG, knowledge graphs for RAG, questions like "what are the main themes across all documents", or when vector RAG fails on questions that need the whole dataset.
level: 4-advanced
---

# 11 - GraphRAG

**Simple-ah solna:** Vector RAG top-k thundu mattum paakkum. "Indha 500 documents-la main themes enna?" maathiri **muzhu dataset** pathi kelvi-na adhu fail aagum. GraphRAG LLM-aal entities + relationships graph build panni, communities-ku summary vechchu, adhula irundhu answer solludhu.

## How it works (Microsoft GraphRAG)
1. LLM extracts entities and relationships from all text -> knowledge graph.
2. Detect hierarchical **communities** of densely connected nodes.
3. LLM writes a summary per community (a pre-computed overview of the data).
4. Global question -> map-reduce: ask each group of community summaries, then combine the partial answers.

## When it earns its cost
Use when: questions are corpus-wide (themes, trends, "across all"), relationship-heavy (people/orgs/events), or sense-making style.
Skip when: questions are lookups answerable from 1-3 chunks. Hybrid + rerank + contextual retrieval is far cheaper and usually enough.

## Evidence and caveats
- Microsoft reports GraphRAG beating naive RAG on comprehensiveness and diversity (about 70-80% win rate) in LLM-judged tests on podcast and news datasets. Judge-based results on two datasets: validate on your own data.
- The project warns that **indexing can be expensive**; read the docs and start small. Prompt tuning is strongly recommended.
- It is a methodology demo repo, not an officially supported Microsoft product.

## Getting started plan
1. Pick 20-50 documents and 10 global questions you actually care about.
2. Run the GraphRAG CLI quickstart on that small slice; note token cost.
3. Compare against your best vector pipeline on those questions (skill 07).
4. Scale only if the win is real.

## Done when
- [ ] You have a list of global questions that vector RAG demonstrably fails on.
- [ ] Cost per indexed document measured on a small slice.
- [ ] Decision documented: adopt / skip.

## Learn more
- Repo: https://github.com/microsoft/graphrag
- Microsoft Research blog: https://www.microsoft.com/en-us/research/blog/graphrag-new-tool-for-complex-data-discovery-now-on-github/
