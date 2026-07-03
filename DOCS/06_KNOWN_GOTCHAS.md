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

---

## 3. Logfire Initialization Order (The "Poisoning" Bug)

**The Issue:**
If any module calls `logfire.info()`/`logfire.span()` *before*
`logfire.configure()` has run, Logfire's internal state becomes "poisoned"
for that process — it silently enters a no-op mode and discards all
subsequent traces, even if you call `.configure()` later. Importing
`app.config.settings` (or anything that transitively imports it) at the top
of `app/main.py` risks pulling in a module with a module-level Logfire call
before configuration happens.

**The Solution:**
`app/main.py` bypasses `app.config` entirely at the very top of the file —
it loads `.env` and calls `logfire.configure(token=os.getenv("LOGFIRE_TOKEN"))`
using raw `os.getenv()`, before importing anything else from `app`.

```python
# app/main.py
import logfire
import os
from dotenv import load_dotenv

load_dotenv()
logfire.configure(token=os.getenv("LOGFIRE_TOKEN"))

# Safe to import the rest of the application now!
from app.agents.graph import rag_agent
```

## 4. No Conversation Memory Yet

**The Issue:**
`app/agents/graph.py` compiles the `StateGraph` with `workflow.compile()`
and no checkpointer. Every `/query` call builds a brand-new `initial_state`
and there is nothing tying separate HTTP requests together — the Planner's
"answer from conversation history" branch only has access to whatever
`messages` were sent in *that one request*.

**Why it's left this way for now:** introducing `MemorySaver` and a
`thread_id` at the same time as the Planner/Retriever/Responder split would
be two lessons at once. Memory arrives in the next stage, once the basic
flow is understood.
