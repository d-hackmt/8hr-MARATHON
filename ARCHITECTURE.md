# Architecture — Built Up Stage by Stage

This diagram grows one subgraph at a time as each lesson is introduced.

## Stage 1 — Data Ingestion

```mermaid
graph LR
    subgraph INGEST ["📥 Ingestion Pipeline"]
        direction TB
        LOADER["Document Loaders\nPDF · HTML · DOCX · PPTX · TXT"]
        PARSED[("📁 processed_data/\nLocal JSON Chunks")]
        EMB["🔢 Gemini Embeddings\ngemini-embedding-2-preview · 3072-dim"]
    end
    QD[("🗄️ Qdrant Cloud\nVector DB")]

    LOADER --> PARSED
    PARSED --> EMB
    EMB --> QD

    classDef ingest fill:#4F46E5,stroke:#3730A3,color:#fff
    classDef db fill:#059669,stroke:#065F46,color:#fff
    class LOADER,PARSED,EMB ingest
    class QD db
```

`DATA/true_data/` is parsed by extension-specific loaders, chunked into
~1500-character paragraph-aware pieces, saved locally, embedded with
Gemini, and upserted into a Qdrant Cloud collection.

The next stage (`stage-2-basic-rag`) adds the Agent Engine and Interface
subgraphs that read from this vector store.
