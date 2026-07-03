# Architecture — Built Up Stage by Stage

This is the complete system, built one lesson at a time across 6 branches.

## Stage 6 — Evals (final)

```mermaid
graph TB

    subgraph UI ["1. User Interface"]
        direction LR
        CHAT["Streamlit Chat UI"]
        EAPP["Streamlit Eval App"]
    end

    subgraph SAFETY ["2. API + Safety Gate"]
        direction LR
        API["⚡ FastAPI  /query"]
        GR{"🛡️ NeMo Guardrails\nBlocks · Jailbreak · Off-topic · Injection"}
    end

    subgraph AGENT ["3. Agent Engine — LangGraph"]
        direction LR
        PL["🗺️ Planner Node\nIntent Classification"]
        RT["🔍 Retriever Node\nVector Search"]
        RS["💬 Responder Node\nAnswer Generation"]
        MEM[("💾 MemorySaver\nConversation History")]
    end

    subgraph KNOWLEDGE ["4. Knowledge & LLMs"]
        direction LR
        QD[("🗄️ Qdrant Cloud\nVector DB")]
        FR["⚡ FlashRank\nLocal Reranker"]
        PK["🔀 Portkey Gateway\nRouting + Fallback + Cache"]
        G1["🦙 Groq Primary\nLlama 3.3 · 70B"]
        G2["🦙 Groq Fallback\nLlama 3.1 · 8B"]
    end

    subgraph INGEST ["5. Data Ingestion"]
        direction LR
        LOAD["Document Loaders\nPDF · HTML · DOCX · PPTX · TXT"]
        PROC[("📁 processed_data/\nLocal JSON Chunks")]
        EMB["🔢 Gemini Embeddings\ngemini-embedding-2-preview · 3072-dim"]
    end

    subgraph EVALS ["6. Evaluation Suite — RAGAS"]
        direction LR
        GD[("📋 Golden Dataset\n15 RAG Samples · 6 Guardrail Tests")]
        RAGAS["RAGAS Metrics\nFaithfulness · Relevancy · Precision\nRecall · Correctness"]
        TC["Tool Correctness\nJaccard · Zero LLM Cost"]
        JG["⚖️ Judge LLM\nGroq · Separate Key"]
    end

    CHAT -->|user query| API
    EAPP -->|phase 1 query| API
    API --> GR
    GR -->|"❌ blocked"| CHAT
    GR -->|"✅ pass"| PL
    PL -->|"technical"| RT
    PL -->|"conversational"| RS
    RT --> QD
    QD --> FR
    FR --> RS
    RS --> PK
    PL --> PK
    PK --> G1
    PK -.->|"fallback"| G2
    RS -.-> MEM
    MEM -.-> PL

    LOAD --> PROC
    PROC --> EMB
    EMB --> QD

    GD --> RAGAS
    GD --> TC
    RAGAS --> JG

    classDef ui        fill:#2563EB,stroke:#1E40AF,color:#fff
    classDef safety    fill:#DC2626,stroke:#991B1B,color:#fff
    classDef agent     fill:#7C3AED,stroke:#5B21B6,color:#fff
    classDef knowledge fill:#D97706,stroke:#92400E,color:#fff
    classDef ingest    fill:#4F46E5,stroke:#3730A3,color:#fff
    classDef evals     fill:#DB2777,stroke:#9D174D,color:#fff
    classDef memory    fill:#6D28D9,stroke:#4C1D95,color:#fff

    class CHAT,EAPP ui
    class API,GR safety
    class PL,RT,RS agent
    class QD,FR,PK,G1,G2 knowledge
    class LOAD,PROC,EMB ingest
    class GD,RAGAS,TC,JG evals
    class MEM memory
```

Every subgraph above was introduced one lesson at a time:
- Ingestion — Stage 1
- Agent Engine (no rerank/memory) — Stage 2
- FlashRank + MemorySaver — Stage 3
- Safety Gate — Stage 4
- LLM Gateway — Stage 5
- Evaluation Suite — Stage 6 (this branch)
