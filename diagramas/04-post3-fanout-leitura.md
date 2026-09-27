# Post 3 — Finalização proativa + fan-out de leitura

```mermaid
flowchart LR
    CONS["Consumers<br/>(cálculo + finalização)"] --> MV[("Materialized View<br/>boleto/PIX pronto")]
    MV --> SITE["Site"]
    MV --> APP["App · novo ·"]
    MV --> WA["WhatsApp · novo ·"]

    classDef novo fill:#ffe3e3,stroke:#d33,color:#900,stroke-width:2px
    class MV,APP,WA novo
```

Segunda fatia: o mesmo consumer do post 2 passa a acionar também a Procedure
de Finalização (proativamente, uma vez por fatura), grava o resultado numa
Materialized View, e três canais independentes passam a ler dali.
