# Architecture — Built Up Stage by Stage

This diagram grows one subgraph at a time as each lesson is introduced.

## Stage 4 — Guardrails

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

    CHAT -->|query| API
    API --> GR
    GR -->|"❌ blocked"| CHAT
    GR -->|"✅ pass"| PL
    PL -->|technical| RT
    PL -->|conversational| RS
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
    class API,GR safety
    class PL,RT,RS agent
    class QD,FR knowledge
    class MEM memory
```

The Guardrails gate sits in `main.py`, not inside the LangGraph — it's a
pre-graph HTTP-layer check on the raw user message, called once before
`rag_agent.invoke(...)`. It is input-only: the final LLM answer is never
re-checked.

The next stage (`stage-5-llm-gateway`) adds an LLM Gateway subgraph that
the Planner and Responder route through — but not the guardrails
classifier, which intentionally stays on a direct Groq call.
