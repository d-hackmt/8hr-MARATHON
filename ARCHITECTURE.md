# Architecture — Built Up Stage by Stage

This diagram grows one subgraph at a time as each lesson is introduced.

## Stage 3 — Reranking + Memory

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
        RT["🔍 Retriever Node\nVector Search"]
        RS["💬 Responder Node\nAnswer Generation (direct Groq)"]
        MEM[("💾 MemorySaver\nConversation History")]
    end

    subgraph KNOWLEDGE ["4. Knowledge"]
        direction LR
        QD[("🗄️ Qdrant Cloud\nVector DB")]
        FR["⚡ FlashRank\nLocal Reranker"]
    end

    CHAT -->|query + thread_id| API
    API --> PL
    PL -->|conversational| RS
    PL -->|technical| RT
    RT --> QD
    QD --> FR
    FR --> RS
    RS --> API
    API --> CHAT
    RS -.-> MEM
    MEM -.-> PL

    classDef ui        fill:#2563EB,stroke:#1E40AF,color:#fff
    classDef safety    fill:#DC2626,stroke:#991B1B,color:#fff
    classDef agent     fill:#7C3AED,stroke:#5B21B6,color:#fff
    classDef knowledge fill:#D97706,stroke:#92400E,color:#fff
    classDef memory    fill:#6D28D9,stroke:#4C1D95,color:#fff

    class CHAT ui
    class API safety
    class PL,RT,RS agent
    class QD,FR knowledge
    class MEM memory
```

Two additions on top of Stage 2: the Retriever now routes through
**FlashRank** before reaching the Responder, and the graph carries a
**MemorySaver** checkpoint keyed by `thread_id`. Still no LLM gateway, still
no guardrails.

The next stage (`stage-4-guardrails`) adds a Safety Gate subgraph in front
of the Agent Engine.
