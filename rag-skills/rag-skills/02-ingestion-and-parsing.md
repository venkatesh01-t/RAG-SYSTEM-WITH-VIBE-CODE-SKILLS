---
name: rag-ingestion-and-parsing
description: How to load and parse documents (PDF, DOCX, HTML, scanned files, tables) into clean text with metadata for a RAG system, using Docling, plus deduplication and incremental updates. Use this skill whenever the user is building the data pipeline of a RAG system, has messy PDFs/tables/scans, asks why retrieval returns garbage, or needs a metadata schema.
level: 1-foundation
---

# 02 - Ingestion and Parsing

**Simple-ah solna:** Garbage in, garbage out. PDF-la irundhu text sariyaa edukkalana, evlo periya model vachaalum hallucination dhaan. Mudhal vela = clean text + nalla metadata.

## Steps
1. **Inventory sources**: list every file type, owner, update frequency, trust level.
2. **Parse to a structured format** (Markdown preferred: headings and tables survive).
3. **Clean**: remove headers/footers/page numbers, fix hyphenation, drop boilerplate.
4. **Attach metadata** to every document and later every chunk.
5. **Deduplicate** (hash of normalized text).
6. **Make it incremental**: re-index only changed files.

## Parsing with Docling
Docling handles PDF, DOCX, PPTX, HTML, images with layout, reading order, table structure and OCR, and exports Markdown/JSON.
```python
# pip install docling
from docling.document_converter import DocumentConverter
conv = DocumentConverter()
result = conv.convert("report.pdf")            # local path or URL
markdown = result.document.export_to_markdown()
```
Always open 10-20 parsed outputs and read them. If tables are broken here, no later step can fix it.

## Metadata schema (minimum)
```json
{
  "doc_id": "stable id",
  "source": "file path or URL",
  "title": "...",
  "section": "heading path",
  "page": 12,
  "version": "2026-11",
  "content_hash": "sha256",
  "access": ["team-a"],
  "ingested_at": "ISO date"
}
```
`access` matters later for security (skill 12). `version` + `content_hash` enable incremental updates and rollbacks.

## Incremental update pattern
```python
import hashlib
def sha(text): return hashlib.sha256(text.encode()).hexdigest()
# if sha(new_text) == stored_hash -> skip; else delete old chunks for doc_id, re-chunk, re-embed
```

## Common mistakes
- Using a naive PDF text extractor on tables/multi-column pages.
- Dropping page/section info, which makes citations impossible later.
- Indexing outdated or duplicate versions of the same document.

## Done when
- [ ] 20 random parsed pages manually checked.
- [ ] Every document has the metadata fields above.
- [ ] Re-running ingestion on unchanged files does nothing.

## Learn more
- Docling: https://github.com/DS4SD/docling and paper https://arxiv.org/abs/2408.09869
- Tutorial using local parsing + LangChain agent: https://www.freecodecamp.org/news/build-a-private-rag-qa-ai-agent-for-your-documents-using-langchain/
