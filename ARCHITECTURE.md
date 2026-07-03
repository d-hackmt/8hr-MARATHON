# Architecture — Built Up Stage by Stage

This diagram grows one subgraph at a time as each lesson is introduced. On
this branch, nothing has been built yet.

```mermaid
graph TB
    A["📦 Sample Data Only\nDATA/true_data/"]
    classDef empty fill:#6B7280,stroke:#374151,color:#fff
    class A empty
```

The next stage (`stage-1-ingestion`) adds the first real subgraph: the
document ingestion pipeline.
