# Enterprise Agentic RAG (Scalable Pipeline)

This repository was taught **incrementally, one git branch per lesson**.
This final branch (`stage-6-evals`) has everything: a production-grade,
enterprise-level RAG system built with **LangGraph**, **Portkey LLM
Gateway**, and **Gemini Embeddings**, guarded by **NeMo Guardrails**, and
measured by a **RAGAS** evaluation suite.

## Lesson roadmap

| Stage | Branch | What you'll build |
|---|---|---|
| 1 | `stage-1-ingestion` | Parse local documents, chunk them, embed them, and index them into a vector database |
| 2 | `stage-2-basic-rag` | A minimal FastAPI + LangGraph RAG agent — no reranking, no memory yet |
| 3 | `stage-3-rerank-memory` | Add a local semantic reranker and multi-turn conversation memory |
| 4 | `stage-4-guardrails` | Add an input safety gate that blocks off-topic and jailbreak attempts |
| 5 | `stage-5-llm-gateway` | Route all LLM calls through an LLM gateway with automatic fallback and caching |
| **6** | **`stage-6-evals`** ← you are here | Add a RAGAS-based evaluation suite to measure the whole system |

## Key Features

- **Agentic Intelligence**: LangGraph for cyclic reasoning, multi-step planning, and conversation memory.
- **Guardrails**: NeMo Guardrails gate blocks off-topic, jailbreak, and injection inputs before any retrieval.
- **LLM Gateway**: Portkey routes all LLM calls with automatic fallback between primary and backup Groq keys.
- **Enterprise Search**: Qdrant Cloud for high-performance vector search + FlashRank for local semantic reranking.
- **Gemini Embeddings**: Google `gemini-embedding-2-preview` (3072-dim) via `langchain-google-genai`.
- **Local Document Parsing**: PDF, HTML, TXT, DOCX, PPTX parsed entirely on-device — no external OCR service.
- **Observability**: Full trace nesting with **Pydantic Logfire** and **LangSmith** across every agent node.
- **Evaluation Suite**: RAGAS-powered eval pipeline (6 metrics) with a dedicated Streamlit demo app.

---

## Stage 6 — Evals

Adds an offline evaluation suite that queries the running system from the
outside and scores it:

- **`evals/pipeline.py`** — runs all 15 golden questions against the live
  `/query` endpoint and records `actual_response`/`actual_contexts`/`actual_tools_called`.
- **`evals/metrics.py`** — 6 metrics: RAGAS Faithfulness, Answer Relevancy,
  Context Precision, Context Recall, Answer Correctness (judged by a
  separate `JUDGE_GROQ`-keyed LLM), plus a zero-LLM-cost **Tool
  Correctness** Jaccard metric.
- **`evals/guardrails_eval.py`** — runs the 6 guardrail test cases and
  computes TP/TN/FP/FN, precision, recall, and accuracy.
- **`evals/app.py`** — a 3-tab Streamlit demo: Ground Truth, Live Pipeline,
  Eval Metrics.

### 1. Install dependencies

```powershell
pip install -r requirements.txt
```

### 2. Configure environment

Add `JUDGE_GROQ` to your `.env` — a separate Groq key so eval runs can't
rate-limit the live app's production key.

### 3. Run data ingestion

```powershell
python -m app.ingestion.processor DATA --wipe
```

### 4. Launch the app

```powershell
# Terminal 1 — FastAPI backend
uvicorn app.main:app --reload --port 8000

# Terminal 2 — Streamlit UI
streamlit run ui/app.py
```

### 5. Run the eval suite

```powershell
# Requires the FastAPI backend running on :8000
streamlit run evals/app.py
```

### Verify it worked

- "Live Pipeline" tab populates all 15 golden questions with real answers.
- "Eval Metrics" tab produces scores for all 6 metrics plus guardrails
  accuracy.
- `evals/guardrails_eval.py`'s 6 test cases compute a full confusion matrix.

---

## Documentation Index

| # | Guide | What it covers |
|---|-------|---------------|
| 01 | [System Overview](DOCS/01_SYSTEM_OVERVIEW.md) | High-level vision and end-to-end flow |
| 02 | [Ingestion Engine](DOCS/02_INGESTION_ENGINE.md) | Document parsing and indexing pipeline |
| 03 | [Node Intelligence](DOCS/03_NODE_INTELLIGENCE.md) | Planner, Retriever, Responder internals |
| 04 | [Observability](DOCS/04_TRACING_AND_OBSERVABILITY.md) | Logfire + LangSmith tracing |
| 05 | [Environment Variables](DOCS/05_ENVIRONMENT_VARIABLES.md) | All env vars and configuration reference |
| 06 | [Known Gotchas](DOCS/06_KNOWN_GOTCHAS.md) | Non-obvious bugs and architectural decisions |
| 07 | [FlashRank Reranking](DOCS/07_FLASHRANK_RERANKING.md) | Local semantic reranker deep-dive |
| 08 | [Guardrails](DOCS/08_GUARDRAILS.md) | NeMo Guardrails implementation |
| 09 | [LLM Gateway](DOCS/09_LLM_GATEWAY.md) | Portkey routing, fallback, and observability |
| 10 | [Evals](DOCS/10_EVALS.md) | RAGAS metrics theory and token budget |
| 11 | [Evals Pipeline](DOCS/11_EVALS_PIPELINE.md) | Live eval pipeline and Streamlit demo |

---

*Built for High-Scale Enterprise Document Intelligence.*
