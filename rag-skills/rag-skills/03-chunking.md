---
name: rag-chunking
description: Chunking strategies for RAG - fixed size with overlap, recursive, structure-aware (Markdown headers), semantic, proposition and parent-child chunking - and how to tune chunk size with evaluation. Use this skill whenever the user splits documents for a RAG pipeline, asks about chunk size or overlap, or sees answers that miss context or retrieve half-sentences.
level: 1-foundation
---

# 03 - Chunking

**Simple-ah solna:** Periya document-ah LLM-ku mulusaa kudukka mudiyaadhu, so chinna chinna thundu-aa udaikkanum. Aana thappaa udaicha meaning poiduchu, retrieval fail aagum.

## Strategies (low -> high effort)
| Strategy | Idea | Use when |
|---|---|---|
| Fixed size + overlap | Cut every N chars/tokens | Quick baseline only |
| Recursive | Split on paragraphs, then sentences, then words | Good default |
| Structure-aware | Split by Markdown headers / sections, keep the heading path | Docs with real structure (manuals, policies) |
| Semantic | Split where embedding similarity drops | Long unstructured prose |
| Proposition | Rewrite into atomic facts | Fact-dense text, high precision needs |
| Parent-child | Search small chunks, return the bigger parent for context | Precise match + full context |

## Starting point (tune it, do not trust it)
- 300-800 tokens, 10-15% overlap is a sane first guess.
- The LangChain RAG tutorial uses 1000 characters with 200 overlap (characters, not tokens).
- Prepend the heading path to every chunk ("Policy > Leave > Sick leave") so a chunk is self-explanatory.

```python
# pip install langchain-text-splitters
from langchain_text_splitters import RecursiveCharacterTextSplitter, MarkdownHeaderTextSplitter

headers = [("#", "h1"), ("##", "h2"), ("###", "h3")]
by_header = MarkdownHeaderTextSplitter(headers_to_split_on=headers).split_text(markdown)

splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=150)
chunks = splitter.split_documents(by_header)   # keeps header metadata
```

## Tune with a loop, not opinions
1. Build the golden set first (skill 07).
2. For chunk_size in [300, 500, 800, 1200]: re-index, measure retrieval hit-rate@k and context recall.
3. Keep the winner. Re-check when the document types change.

## Common mistakes
- One chunk size for every document type.
- Splitting tables or code blocks in half.
- Huge chunks: precision drops and the prompt gets noisy.
- Tiny chunks with no context: add heading path or use contextual retrieval (skill 09).

## Done when
- [ ] Chunks carry source, page, heading path.
- [ ] You compared at least 3 chunk sizes on your golden set.
- [ ] No table or code block is cut mid-way.

## Learn more
- RAG_Techniques notebooks: "Optimizing Chunk Sizes", "Proposition Chunking": https://github.com/NirDiamant/RAG_Techniques
- LangChain RAG tutorial (splitter usage): https://docs.langchain.com/oss/python/langchain/rag
- Anthropic Contextual Retrieval (fixes lost context in chunks): https://www.anthropic.com/engineering/contextual-retrieval
