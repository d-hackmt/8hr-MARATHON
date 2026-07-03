# Enterprise Agentic RAG (Taught in Stages)

This repository is taught **incrementally, one git branch per lesson**. Each
branch is a small, runnable step on top of the previous one — nothing is
introduced before it's explained, and nothing is ever re-explained twice.

This branch (`teaching/00-scaffold`) is the empty starting point: just the
raw sample data and project scaffolding. There is no application code yet.

## Lesson roadmap

| Stage | Branch | What you'll build |
|---|---|---|
| 1 | `stage-1-ingestion` | Parse local documents, chunk them, embed them, and index them into a vector database |
| 2 | `stage-2-basic-rag` | A minimal FastAPI + LangGraph RAG agent — no reranking, no memory yet |
| 3 | `stage-3-rerank-memory` | Add a local semantic reranker and multi-turn conversation memory |
| 4 | `stage-4-guardrails` | Add an input safety gate that blocks off-topic and jailbreak attempts |
| 5 | `stage-5-llm-gateway` | Route all LLM calls through an LLM gateway with automatic fallback and caching |
| 6 | `stage-6-evals` | Add a RAGAS-based evaluation suite to measure the whole system |

Check out the next branch when you're ready to move on:

```powershell
git checkout stage-1-ingestion
```

## What's in this branch

- `DATA/true_data/` — the real sample documents used throughout the course
- `.gitignore`, `requirements.txt`, `.env.example` — empty scaffolding, filled in stage by stage
- `DOCS/05_ENVIRONMENT_VARIABLES.md`, `DOCS/06_KNOWN_GOTCHAS.md` — living reference docs that grow every stage
