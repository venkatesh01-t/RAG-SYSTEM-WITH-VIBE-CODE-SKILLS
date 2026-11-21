---
name: rag-evaluation
description: How to evaluate a RAG system - build a golden question set (including unanswerable questions), measure retrieval (hit-rate, context precision/recall) and generation (faithfulness, answer relevancy, abstention) separately, and use RAGAS or DeepEval in CI. Use this skill whenever the user asks "is my RAG good", wants to measure hallucination, compare chunk sizes/models/prompts, or set quality gates before shipping.
level: 3-reliability
---

# 07 - Evaluation

**Simple-ah solna:** Measure panna mudiyaadhadhai improve panna mudiyaadhu. Mudhala 50-100 kelvi + sariyaana answer ready pannu. Appuram ovvoru maatramum (chunk size, model, prompt) andha set-la test pannu.

## Step 0: Build the golden set BEFORE tuning
Aim for 50-100 rows, from real user questions where possible.
| Type | Why |
|---|---|
| Simple factual | Baseline |
| Exact ID / name / code | Tests keyword side of hybrid |
| Multi-hop (needs 2+ chunks) | Tests recall |
| **Unanswerable** (not in docs) | Tests abstention: the hallucination test |
| Ambiguous / out-of-scope | Tests refusal and clarification |
Each row: `question`, `reference_answer` (or "NOT_IN_DOCS"), `gold_chunk_ids`.

## Metrics: separate retrieval from generation
| Layer | Metric | Meaning |
|---|---|---|
| Retrieval | Hit-rate@k / recall | Did the right chunk appear in top-k? |
| Retrieval | Context precision | Fraction of retrieved chunks that are relevant (and ranked high) |
| Retrieval | Context recall | Did retrieval cover what the reference answer needs? |
| Generation | **Faithfulness** | Supported claims / total claims in the answer (grounded in context) |
| Generation | Answer relevancy | Does the answer address the question? |
| Safety | Correct-abstain rate | On unanswerable questions, how often it refuses |
| Safety | False-abstain rate | On answerable questions, how often it wrongly refuses |

Diagnose by layer: low hit-rate = retrieval problem (skills 03-05, 09); good hit-rate but low faithfulness = prompt/generation problem (06, 08).

## Zero-dependency retrieval check
```python
def hit_rate_at_k(golden, retrieve, k=5):
    hits = 0
    for ex in golden:                       # {"q": ..., "gold_chunk_ids": [...]}
        got = {c["id"] for c in retrieve(ex["q"])[:k]}
        hits += bool(got & set(ex["gold_chunk_ids"]))
    return hits / len(golden)
```

## Tools
- **RAGAS**: faithfulness, answer relevancy, context precision/recall, test-set generation. Follow its current quickstart (API changes between versions).
- **DeepEval**: pytest-style tests; RAG metrics include Answer Relevancy, Faithfulness, Contextual Recall/Precision/Relevancy.
```python
# pip install -U deepeval  (needs a judge LLM key configured)
from deepeval import assert_test
from deepeval.metrics import FaithfulnessMetric
from deepeval.test_case import LLMTestCase

def test_faithful():
    case = LLMTestCase(input=q, actual_output=answer, retrieval_context=[c["text"] for c in chunks])
    assert_test(case, [FaithfulnessMetric(threshold=0.8)])
```

## Rules of the game
1. Change **one** variable at a time; log a results table (run id, change, metrics).
2. LLM judges make mistakes: manually review 20 failures per run.
3. Gate releases in CI: if faithfulness or correct-abstain drops below your threshold, block.
4. Feed production failures back into the golden set (skill 12).

## Done when
- [ ] Golden set with >= 15% unanswerable rows.
- [ ] Baseline numbers recorded for every metric above.
- [ ] One command re-runs the whole evaluation.

## Learn more
- RAGAS: https://github.com/explodinggradients/ragas and https://docs.ragas.io/en/latest/getstarted/rag_evaluation/
- DeepEval: https://github.com/confident-ai/deepeval
- RAG_Techniques evaluation notebooks (DeepEval, GroUSE): https://github.com/NirDiamant/RAG_Techniques
