# ⚠️ Known Gotchas & Architectural Decisions

This document tracks non-obvious platform quirks and explains why the
architecture is designed the way it is. It is a **living reference** — it
grows one entry at a time as each lesson introduces the code that the
gotcha applies to.

---

## 1. Embedding Dimension Is Resolved at Runtime, Not Hardcoded

**The Issue:**
Gemini's `gemini-embedding-2-preview` returns 3072-dim vectors, but if the
Gemini API is unavailable, `app/services/retrieval/embedding.py` silently
falls back to a local `SentenceTransformer` model (768-dim). If the Qdrant
collection were created with a hardcoded dimension, ingestion would fail
with a dimension mismatch the moment the fallback kicked in.

**The Solution:**
`app/ingestion/processor.py` calls `get_embedding_dim()` — which actually
probes whichever embedding backend initialized — immediately before
creating the Qdrant collection, so the collection's vector size always
matches whatever is about to be written to it.

---

## 2. Chunking Is Character-Length + Paragraph-Boundary Only

**The Issue:**
`app/ingestion/chunking/splitter.py`'s `chunk_text()` splits on blank lines
(`"\n\n"`) and greedily accumulates paragraphs up to a 1500-character
ceiling — it does not do token-aware splitting and does not add overlap
between chunks. This is intentional for this stage: it's simple, fast, and
good enough for the document types in `DATA/true_data/`. Keep it in mind if
you swap in documents with very long, unbroken paragraphs.
