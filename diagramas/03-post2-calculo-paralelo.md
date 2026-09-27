# Post 2 — CDC dispara, cálculo roda em paralelo

```mermaid
flowchart LR
    LEG["Legado<br/>status muda: fatura pronta"] -->|"observado por"| CDC["CDC (Debezium)"]
    CDC -->|"publica"| KAFKA{{"Kafka<br/>fatura.pronta"}}
    KAFKA --> CONS["Consumer de Cálculo<br/>· Competing Consumers, N instâncias ·<br/>cada um aciona CalculaFatura,<br/>1 aluno por vez"]

    classDef novo fill:#ffe3e3,stroke:#d33,color:#900,stroke-width:2px
    class CDC,CONS novo
```

Primeira fatia da arquitetura proposta (`02-arquitetura-proposta.md`): só o
gatilho autônomo (CDC) e o paralelismo de `CalculaFatura`. Ainda sem
Consumer de Boleto/NFS-e, Materialized View ou canais novos — isso vem no
post 3.
