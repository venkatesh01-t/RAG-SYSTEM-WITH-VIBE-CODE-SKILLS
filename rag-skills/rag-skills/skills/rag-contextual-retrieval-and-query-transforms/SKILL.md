---
name: rag-contextual-retrieval-and-query-transforms
description: Advanced retrieval boosters for RAG - Anthropic's Contextual Retrieval (contextual embeddings + contextual BM25 with prompt caching), plus query rewriting, multi-query, HyDE, decomposition and conversation condensing. Use this skill whenever baseline retrieval still misses relevant chunks, chunks lose context ("the company", "this quarter"), or user questions are vague or multi-part.
level: 4-advanced
---

# 09 - Contextual Retrieval and Query Transforms

**Simple-ah solna:** Sila chunk-la "company revenue 3% grow aachu" nu irukkum, aana edhu company, edhu quarter nu irukkaadhu. Andha missing context-ah chunk-oda munnaadi serthu embed pannina retrieval nalla aagum.

## Contextual Retrieval (Anthropic)
Idea: for each chunk, have an LLM write a short situating context from the whole document (about 50-100 tokens), **prepend** it to the chunk, then use the enriched text for both embeddings and BM25.
Reported results (Anthropic's tests): top-20 retrieval failure rate fell by 35% with contextual embeddings, 49% with contextual embeddings + contextual BM25, and up to 67% when reranking was added.
Cost control: prompt caching lets you load the document once and reuse it for every chunk of that document.

```python
CONTEXT_PROMPT = """<document>
{doc}
</document>
Here is a chunk from that document:
<chunk>
{chunk}
</chunk>
Write 1-2 short sentences that situate this chunk within the overall document
(company/section/time period/topic) to improve search retrieval. Answer with the context only."""

def contextualize(doc_text, chunk_text):
    msg = client.messages.create(
        model=MODEL, max_tokens=150, temperature=0,
        messages=[{"role": "user", "content": CONTEXT_PROMPT.format(doc=doc_text, chunk=chunk_text)}])
    return msg.content[0].text.strip() + "\n\n" + chunk_text   # embed + BM25 this text
```
(Use a cheaper/smaller model for this indexing job and enable prompt caching; see the Anthropic post and cookbook.)
When to skip: very small corpora where the whole knowledge base fits in the prompt is simpler; Anthropic notes the simple long-prompt approach is fine until the knowledge base outgrows it.

## Query transforms (use when eval shows query-side failures)
| Technique | What it does | Use when |
|---|---|---|
| Rewrite / condense | Turn chat history + follow-up into a standalone query | Multi-turn chat ("what about 2024?") |
| Multi-query | Generate 3-5 paraphrases, retrieve for each, fuse with RRF | Vocabulary mismatch |
| HyDE | LLM writes a hypothetical answer; embed that to search | Short or vague queries |
| Decomposition | Split a complex question into sub-questions | Multi-hop questions |
| Step-back | Ask a more general question first | Questions needing background |
Every transform adds latency and cost and can drift from user intent: keep the **original question** for the final answer and A/B each transform on the golden set.

## Done when
- [ ] Failure analysis shows which queries failed and why before adding a technique.
- [ ] Contextual indexing compared against plain indexing on the golden set.
- [ ] Each query transform kept only if it improved metrics.

## Learn more
- Anthropic, Introducing Contextual Retrieval: https://www.anthropic.com/engineering/contextual-retrieval
- Simon Willison's summary: https://simonwillison.net/2024/Sep/20/introducing-contextual-retrieval
- DataCamp implementation walkthrough: https://www.datacamp.com/tutorial/contextual-retrieval-anthropic
- AWS: contextual retrieval with Bedrock Knowledge Bases: https://aws.amazon.com/blogs/machine-learning/contextual-retrieval-in-anthropic-using-amazon-bedrock-knowledge-bases
- Query-transform notebooks: https://github.com/NirDiamant/RAG_Techniques
