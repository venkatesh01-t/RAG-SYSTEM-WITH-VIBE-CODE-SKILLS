# Research Results and Source Directory

Collected via web search on 2026-11-21. Each link below appeared in search results; open each once to confirm it still loads. "Skill" = which skill file uses it.

## A. Papers (theory, low -> high)
| Resource | URL | Use for | Skill |
|---|---|---|---|
| RAG original paper (Lewis et al., 2020) | https://arxiv.org/abs/2005.11401 | Where RAG started | 01 |
| RAG survey (Gao et al.): Naive/Advanced/Modular RAG, evaluation | https://arxiv.org/abs/2312.10997 | Best overview paper | 01, 08 |
| Survey companion repo | https://github.com/Tongji-KGLLM/RAG-Survey | Resources from the survey authors | 01 |
| MTEB: Massive Text Embedding Benchmark | https://arxiv.org/abs/2210.07316 | Embedding model comparison | 04 |
| Docling technical report | https://arxiv.org/abs/2408.09869 | How the parser works | 02 |
| Self-RAG (Asai et al.) | https://arxiv.org/abs/2310.11511 | Retrieve-on-demand + self-critique | 08, 10 |
| CRAG: Corrective RAG | https://arxiv.org/abs/2401.15884 | Retrieval evaluator + corrective actions | 08, 10 |

## B. Code and open-source projects
| Project | URL | Level | Skill |
|---|---|---|---|
| RAG_Techniques (42+ runnable notebooks, foundational to advanced, evaluation) | https://github.com/NirDiamant/RAG_Techniques | All | 03, 06, 07, 09, 13 |
| Docling (document parsing) | https://github.com/DS4SD/docling | 1 | 02 |
| FlagEmbedding / BGE (embeddings, rerankers) | https://github.com/FlagOpen/FlagEmbedding | 2 | 04, 05 |
| bge-reranker-v2-m3 | https://huggingface.co/BAAI/bge-reranker-v2-m3 | 2 | 05 |
| MTEB code | https://github.com/embeddings-benchmark/mteb | 2 | 04 |
| MTEB leaderboard | https://huggingface.co/spaces/mteb/leaderboard | 2 | 04 |
| pgvector | https://github.com/pgvector/pgvector | 2 | 04, 12 |
| LlamaIndex | https://github.com/run-llama/llama_index | 2-4 | 01, 10 |
| RAGAS (evaluation) | https://github.com/explodinggradients/ragas | 3 | 07 |
| DeepEval (evaluation, pytest style) | https://github.com/confident-ai/deepeval | 3 | 07 |
| Self-RAG code | https://github.com/AkariAsai/self-rag | 4 | 08, 10 |
| CRAG code | https://github.com/HuskyInSalt/CRAG | 4 | 08, 10 |
| Microsoft GraphRAG | https://github.com/microsoft/graphrag | 4 | 11 |
| Langfuse (observability) | https://github.com/langfuse/langfuse | 5 | 12 |

## C. Official documentation and guides
| Doc | URL | Skill |
|---|---|---|
| LangChain: Build a RAG agent | https://docs.langchain.com/oss/python/langchain/rag | 01, 06, 13 |
| LangGraph: Agentic RAG | https://docs.langchain.com/oss/python/langgraph/agentic-rag | 10 |
| LlamaIndex: Introduction to RAG (5 stages) | https://developers.llamaindex.ai/python/framework/understanding/rag/ | 01 |
| LlamaIndex starter tutorial | https://developers.llamaindex.ai/python/framework/getting_started/starter_example/ | 10 |
| LlamaIndex starter (local models) | https://developers.llamaindex.ai/python/framework/getting_started/starter_example_local/ | 13 |
| Qdrant: hybrid queries (prefetch + RRF) | https://qdrant.tech/documentation/search/hybrid-queries/ | 04, 05, 12 |
| Qdrant: hybrid search tuning | https://qdrant.tech/documentation/search-tuning/hybrid-search/ | 05 |
| RAGAS RAG-evaluation quickstart | https://docs.ragas.io/en/latest/getstarted/rag_evaluation/ | 07 |
| Langfuse docs | https://langfuse.com/docs | 12 |

## D. Articles and blogs
| Article | URL | Skill |
|---|---|---|
| Anthropic: Introducing Contextual Retrieval | https://www.anthropic.com/engineering/contextual-retrieval | 03, 05, 09 |
| Simon Willison on Contextual Retrieval | https://simonwillison.net/2024/Sep/20/introducing-contextual-retrieval | 09 |
| DataCamp: Contextual Retrieval implementation | https://www.datacamp.com/tutorial/contextual-retrieval-anthropic | 09 |
| AWS: Contextual retrieval with Bedrock Knowledge Bases | https://aws.amazon.com/blogs/machine-learning/contextual-retrieval-in-anthropic-using-amazon-bedrock-knowledge-bases | 09 |
| Microsoft Research: GraphRAG announcement | https://www.microsoft.com/en-us/research/blog/graphrag-new-tool-for-complex-data-discovery-now-on-github/ | 11 |
| freeCodeCamp: private local RAG agent (LangChain v1, Ollama) | https://www.freecodecamp.org/news/build-a-private-rag-qa-ai-agent-for-your-documents-using-langchain/ | 02, 13 |

## E. How to keep discovering new material
- **arXiv**: search "retrieval-augmented generation", "RAG evaluation", "hallucination mitigation RAG"; sort by newest, read abstracts, then follow "code" links.
- **GitHub**: search `topic:rag`, sort by recently updated; prefer repos with tests, a license, and recent commits. Also search for "awesome RAG" lists.
- **Hugging Face**: model cards for embedding and reranker models; the MTEB leaderboard.
- **Vendor engineering blogs**: Anthropic, Qdrant, Microsoft Research, LlamaIndex, LangChain.
- Always test a new technique on your golden set before adopting it.

## F. Mentioned by name only (links not verified in this pass; search by name)
Chroma and FAISS (local vector stores), rank-bm25 (Python BM25), HyDE and "Lost in the Middle" (query and context-ordering research), sentence-transformers.
