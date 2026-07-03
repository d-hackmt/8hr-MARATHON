# Architecture — Built Up Stage by Stage

This diagram grows one subgraph at a time as each lesson is introduced.

## Stage 5 — LLM Gateway

```mermaid
graph TB

    subgraph UI ["1. User Interface"]
        direction LR
        CHAT["Streamlit Chat UI"]
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

    CHAT -->|query| API
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
    RS --> API
    API --> CHAT

    classDef ui        fill:#2563EB,stroke:#1E40AF,color:#fff
    classDef safety    fill:#DC2626,stroke:#991B1B,color:#fff
    classDef agent     fill:#7C3AED,stroke:#5B21B6,color:#fff
    classDef knowledge fill:#D97706,stroke:#92400E,color:#fff
    classDef memory    fill:#6D28D9,stroke:#4C1D95,color:#fff

    class CHAT ui
    class API,GR safety
    class PL,RT,RS agent
    class QD,FR,PK,G1,G2 knowledge
    class MEM memory
```

This is the full agent-side architecture. `app/guardrails/rails.py`'s
classifier LLM is intentionally left off this Portkey path — it's the one
LLM call in the project that never moves onto the gateway.

The final stage (`stage-6-evals`) adds a RAGAS Evaluation Suite subgraph
that queries this whole system from the outside to measure it.
