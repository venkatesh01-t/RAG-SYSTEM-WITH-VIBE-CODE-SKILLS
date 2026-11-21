---
name: rag-anti-hallucination-guardrails
description: A layered defense to push RAG hallucination as low as possible - retrieval score gate, strict prompting, programmatic citation checks, an LLM faithfulness verifier, CRAG-style document grading, fail-closed abstention, and human escalation. Use this skill whenever the user says the RAG must not hallucinate, needs high-stakes accuracy, asks how to detect or block unsupported answers, or wants a "theriyala" fallback.
level: 3-reliability
---

# 08 - Anti-Hallucination Guardrails

**Simple-ah solna:** 100% zero illa, aana layers vachu romba kammi pannalam. Ovvoru layer-um "idhu doubtful-na answer kudukkaadha, theriyala nu sollu" nu decision edukkum.

## Defense in depth
| Layer | Check | Blocks |
|---|---|---|
| L0 Data | Trusted, current, deduped sources (skill 02) | Wrong/stale facts at the root |
| L1 Retrieval | Hybrid + rerank (skill 05) | Missing/noisy evidence |
| L2 **Score gate** | Reranker score >= threshold, else abstain **before** calling the LLM | Answering with no evidence |
| L3 Prompt | Only-sources rule, citations, refusal path (skill 06) | Outside-knowledge answers |
| L4 Citation check | Cited IDs exist and are non-empty (code) | Fake citations |
| L5 **Verifier** | Second LLM call: is each claim supported by the sources? | Subtle unsupported claims |
| L6 Fallback | Abstain, ask clarifying question, or hand to a human | Everything that failed above |
| L7 Monitoring | Log, sample, review (skill 12) | Drift |

## L2: score gate (calibrate on your golden set)
```python
def gate(reranked, min_score):          # reranked: [{"text":..., "score":...}, ...]
    return [c for c in reranked if c["score"] >= min_score]
```
Plot reranker scores for answerable vs unanswerable golden questions; choose a threshold that separates them. Do not copy a number from a blog.

## L5: faithfulness verifier
```python
VERIFY_SYSTEM = """You are a strict fact checker. Given SOURCES and an ANSWER:
1. Split the ANSWER into atomic claims.
2. For each claim decide if the SOURCES explicitly support it.
Reply JSON only: {"claims":[{"claim":"...","supported":true|false}]}"""

def verify(answer_text, chunks):
    ctx = "\n\n".join(f"[S{i+1}] {c['text']}" for i, c in enumerate(chunks))
    msg = client.messages.create(
        model=MODEL, max_tokens=800, temperature=0, system=VERIFY_SYSTEM,
        messages=[{"role": "user", "content": f"SOURCES:\n{ctx}\n\nANSWER:\n{answer_text}"}])
    try:
        claims = json.loads(msg.content[0].text)["claims"]
    except Exception:
        return False                    # cannot verify -> fail closed
    return bool(claims) and all(c["supported"] for c in claims)
```
Final flow: `gate -> generate -> citations_ok -> verify -> else ABSTAIN`. (Use `client`, `MODEL`, `ABSTAIN` from skill 06.)

## Self-correcting retrieval (research ideas)
- **CRAG** (Corrective RAG): a lightweight evaluator scores the retrieved documents and triggers one of three actions (Correct / Incorrect / Ambiguous), with web search as a fallback for the latter two.
- **Self-RAG**: trains a model to decide when to retrieve and to critique its own output with reflection tokens. Needs special training; most projects approximate the idea with prompt-based grading (skill 10).

## Know the residual risk
- **Faithful is not the same as true**: if the source document is wrong, a faithful answer is wrong. Fix at L0.
- LLM verifiers can miss errors or reject good answers: measure both on the golden set (skill 07).
- Retrieved documents can contain instructions (prompt injection): treat as data (skill 12).
- High-stakes domains (medical, legal, finance): keep a human in the loop and show citations to the user.

## Done when
- [ ] Threshold calibrated; correct-abstain and false-abstain rates both reported.
- [ ] Verifier blocks the planted-error test cases you added to the golden set.
- [ ] Every failure path ends in a clear refusal message, never a guess.

## Learn more
- CRAG paper: https://arxiv.org/abs/2401.15884 (code: https://github.com/HuskyInSalt/CRAG)
- Self-RAG paper: https://arxiv.org/abs/2310.11511 (code: https://github.com/AkariAsai/self-rag)
- RAG survey (faithfulness and robustness discussion): https://arxiv.org/abs/2312.10997
