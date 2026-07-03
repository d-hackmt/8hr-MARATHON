# Enterprise Agentic RAG (Taught in Stages)

This repository is taught **incrementally, one git branch per lesson**. Each
branch is a small, runnable step on top of the previous one.

## Lesson roadmap

| Stage | Branch | What you'll build |
|---|---|---|
| **1** | **`stage-1-ingestion`** ← you are here | Parse local documents, chunk them, embed them, and index them into a vector database |
| 2 | `stage-2-basic-rag` | A minimal FastAPI + LangGraph RAG agent — no reranking, no memory yet |
| 3 | `stage-3-rerank-memory` | Add a local semantic reranker and multi-turn conversation memory |
| 4 | `stage-4-guardrails` | Add an input safety gate that blocks off-topic and jailbreak attempts |
| 5 | `stage-5-llm-gateway` | Route all LLM calls through an LLM gateway with automatic fallback and caching |
| 6 | `stage-6-evals` | Add a RAGAS-based evaluation suite to measure the whole system |

---

## Stage 1 — Data Ingestion

Turns raw documents in `DATA/true_data/` into searchable vectors in Qdrant.
There is no API and no agent yet — just a standalone CLI pipeline.

```
DATA/true_data/*  →  loader (per file type)  →  chunker  →  processed_data/*.json
                                                          ↘  Gemini embeddings  →  Qdrant Cloud
```

### Project structure

```text
├── app/
│   ├── config.py          # Centralized environment variable management
│   ├── ingestion/
│   │   ├── loaders/       # Local parsers — PDF (pypdf/pdfplumber), HTML, TXT, DOCX/PPTX
│   │   ├── chunking/      # Paragraph-based text splitter (1500 char max)
│   │   └── processor.py   # CLI entrypoint — parse, chunk, embed, index
│   └── services/
│       └── retrieval/
│           └── embedding.py   # Gemini gemini-embedding-2-preview (3072-dim), local fallback
├── DATA/true_data/        # Sample documents (6 files)
└── processed_data/        # Auto-generated — parsed & chunked JSON output per document
```

### 1. Install dependencies

```powershell
python -m venv tenvv
.\tenvv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure environment

Copy `.env.example` to `.env` and fill in `GEMINI_API_KEY`, `QDRANT_API_KEY`,
`QDRANT_CLUSTER_ENDPOINT` (and `LOGFIRE_TOKEN` if you want tracing).

### 3. Run ingestion

```powershell
python -m app.ingestion.processor DATA/true_data true --wipe
```

This parses all 6 files in `DATA/true_data/`, chunks them (1500 chars,
paragraph-aware), writes metadata to `processed_data/true/*.json`, embeds
each chunk with Gemini, and upserts the vectors into a Qdrant collection
named `enterprise_rag`. Pass `--wipe` to drop and recreate the collection;
omit it to append.

### Verify it worked

- `processed_data/true/` should contain 6 JSON files.
- The Qdrant collection's point count should equal the total chunk count
  across those 6 files (check via the Qdrant Cloud console, or
  `QdrantClient(...).count(collection_name="enterprise_rag")`).

Next: check out `stage-2-basic-rag` to turn this indexed data into an
answerable RAG agent.
