---
name: rag-grounded-generation
description: How to write the generation step of a RAG system so answers stay grounded - strict system prompt, source-tagged context, mandatory citations, an explicit "I don't know" path, structured JSON output, temperature 0, and a Claude API code skeleton. Use this skill whenever the user designs the RAG prompt, wants citations, wants the model to refuse when context is missing, or asks how to stop the LLM using outside knowledge.
level: 2-core
---

# 06 - Grounded Generation

**Simple-ah solna:** Retrieval sariyaa irundhaalum, prompt weak-ah irundha LLM "helpful-ah" guess pannum. Prompt-la "context-la illana theriyala nu sollu" nu strict-ah solli, citation compulsory aakkanum.

## Rules for the prompt
1. Answer **only** from the provided sources.
2. Every claim cites a source ID.
3. If sources lack the answer, return a fixed refusal. Give the model a legitimate exit.
4. Output machine-checkable JSON so code can validate it.
5. Temperature 0 (if the model/API accepts it).
6. Fewer, better chunks beat more chunks. Put the most relevant first.

## Skeleton (Anthropic SDK)
```python
# pip install anthropic   (reads ANTHROPIC_API_KEY from env)
import json, anthropic
client = anthropic.Anthropic()
MODEL = "claude-sonnet-5-5"   # confirm current model names in the Anthropic docs

ABSTAIN = {"status": "insufficient_context",
           "answer": "I don't have enough information in the provided documents.",
           "citations": []}

SYSTEM = """You answer questions using ONLY the sources provided inside <sources>.
Rules:
1. Use only facts stated in the sources. Never use outside knowledge.
2. Cite every claim with source ids like S1, S2.
3. If the sources do not contain the answer, set status to "insufficient_context".
4. Reply with JSON only, no extra text:
{"status": "answered" | "insufficient_context", "answer": "...", "citations": ["S1"]}"""

def generate(question, chunks):
    ctx = "\n\n".join(
        f"[S{i+1}] source={c['source']} page={c.get('page','?')}\n{c['text']}"
        for i, c in enumerate(chunks))
    msg = client.messages.create(
        model=MODEL, max_tokens=800, temperature=0, system=SYSTEM,
        messages=[{"role": "user",
                   "content": f"<sources>\n{ctx}\n</sources>\n\nQuestion: {question}"}])
    try:
        return json.loads(msg.content[0].text)
    except (json.JSONDecodeError, IndexError):
        return ABSTAIN          # fail closed, never fail open
```

## Citation validation (code, not vibes)
```python
def citations_ok(out, n_sources):
    if out.get("status") != "answered":
        return True
    ids = out.get("citations", [])
    valid = {f"S{i}" for i in range(1, n_sources + 1)}
    return bool(ids) and set(ids) <= valid
```
If this returns False, return `ABSTAIN`. Stronger version in skill 08 (verifier).

## Common mistakes
- "Use the context to answer" with no refusal path: the model will guess.
- Free-text answers you cannot parse or validate.
- Mixing instructions and retrieved text with no delimiters (use tags like `<sources>`). Retrieved text is **data**, never instructions (prompt-injection defense, skill 12).
- Parsing failure handled by returning the raw text.

## Done when
- [ ] Unanswerable test questions return the refusal, not a guess.
- [ ] Every answered response has valid citations.
- [ ] Parse errors fail closed.

## Learn more
- LangChain RAG agent vs RAG chain: https://docs.langchain.com/oss/python/langchain/rag
- RAG_Techniques "Reliable RAG": https://github.com/NirDiamant/RAG_Techniques
