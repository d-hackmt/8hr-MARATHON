# Architecture — Built Up Stage by Stage

This diagram grows one subgraph at a time as each lesson is introduced.

## Stage 2 — Basic RAG (no reranking, no memory)

```mermaid
graph TB

    subgraph UI ["1. User Interface"]
        direction LR
        CHAT["Streamlit Chat UI"]
    end

    subgraph SAFETY ["2. API"]
        direction LR
        API["⚡ FastAPI  /query"]
    end

    subgraph AGENT ["3. Agent Engine — LangGraph"]
        direction LR
        PL["🗺️ Planner Node\nIntent Classification (direct Groq)"]
        RT["🔍 Retriever Node\nVector Search Only"]
        RS["💬 Responder Node\nAnswer Generation (direct Groq)"]
    end

    subgraph INGEST ["4. Ingestion (from Stage 1)"]
        direction LR
        QD[("🗄️ Qdrant Cloud\nVector DB")]
    end

    CHAT -->|query| API
    API --> PL
    PL -->|conversational| RS
    PL -->|technical| RT
    RT --> QD
    QD --> RT
    RT --> RS
    RS --> API
    API --> CHAT

    classDef ui      fill:#2563EB,stroke:#1E40AF,color:#fff
    classDef safety  fill:#DC2626,stroke:#991B1B,color:#fff
    classDef agent   fill:#7C3AED,stroke:#5B21B6,color:#fff
    classDef ingest  fill:#4F46E5,stroke:#3730A3,color:#fff

    class CHAT ui
    class API safety
    class PL,RT,RS agent
    class QD ingest
```

No reranking, no memory, no guardrails, no LLM gateway yet — the Planner
and Responder nodes call Groq directly. Compare this diagram to Stage 1's:
the Ingestion subgraph didn't change, we just added a way to query it.

The next stage (`stage-3-rerank-memory`) adds a FlashRank reranker between
Retriever and Responder, and a MemorySaver checkpoint on the graph.
