# Post 3 — Boleto e NFS-e isolados, fan-out de leitura

```mermaid
flowchart LR
    K2{{"Kafka<br/>fatura.calculada"}} --> CBOL["Consumer de Boleto<br/>· Sistema de Pagamentos ·"]
    K2 --> CNFE["Consumer de NFS-e<br/>· Sistema Prefeitura ·<br/>isolado do boleto"]
    CBOL --> MV[("Materialized View")]
    CNFE --> MV
    MV --> SITE["Site"]
    MV --> APP["App · novo ·"]
    MV --> WA["WhatsApp · novo ·"]

    classDef novo fill:#ffe3e3,stroke:#d33,color:#900,stroke-width:2px
    class CBOL,CNFE,MV,APP,WA novo
```

Segunda fatia: `GeraFaturaFinal` deixa de ser um bloco só e vira dois
consumers independentes — Boleto e NFS-e, cada um falando só com o terceiro
dele. Os dois gravam na Materialized View, e três canais (site, app,
WhatsApp) passam a só ler.
