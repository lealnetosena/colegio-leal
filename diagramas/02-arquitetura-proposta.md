# Arquitetura proposta — componentes novos

> Resolve P1 e P2 (ver `01-arquitetura-atual.md`), já preparada pra P3 (canais)
> e P4 (aquisição). Componentes novos destacados em vermelho.

```mermaid
flowchart LR
    LEG["Legado<br/>Aplicação + Procedures + Tabela<br/>· particionamento em paralelo ·"] -->|"status muda:<br/>fatura pronta"| CDC["CDC<br/>(Debezium)"]
    CDC -->|"publica"| KAFKA{{"Kafka<br/>fatura.pronta"}}
    KAFKA --> CONS["Consumers de Processamento<br/>· Competing Consumers ·<br/>N instâncias, chamam cálculo +<br/>finalização do legado, 1 aluno por vez"]
    CONS --> MV[("Materialized View<br/>boleto/PIX pronto pra leitura")]
    MV --> SITE["Site"]
    MV --> APP["App · novo ·"]
    MV --> WA["WhatsApp · novo ·"]

    ESC2["Escola adquirida<br/>(legado próprio)"] --> ACL["Anti-Corruption<br/>Layer"]
    ACL -->|"traduz pro mesmo formato"| KAFKA

    classDef novo fill:#ffe3e3,stroke:#d33,color:#900,stroke-width:2px
    class CDC,CONS,MV,ACL novo
```

## Componentes novos (vermelho)

| Componente | Papel | Resolve |
|---|---|---|
| **CDC (Debezium)** | Observa o legado (log de transação), dispara o evento sozinho, sem tocar em código | Gatilho autônomo |
| **Kafka** | Transporta o sinal `fatura.pronta`, de 2 origens (legado próprio + escola adquirida) | Backbone do desacoplamento |
| **Consumers de Processamento** | N instâncias; cada uma aciona a procedure de cálculo e, em seguida, a de finalização, sempre 1 aluno por vez, em paralelo — não é mais um laço só, serial | **P1b** (fim do RBAR) e **P2** (finalização passa a ser proativa, não por clique) |
| **Materialized View** | Guarda o boleto/PIX já pronto | **P2** (site nunca mais aciona a procedure/tabela grande na hora) |
| **App / WhatsApp** | Só leem a Materialized View — chegam de graça, sem tocar em nada existente | **P3** |
| **Anti-Corruption Layer** | Traduz o formato da escola adquirida pro mesmo evento `fatura.pronta` | **P4** |

## O que fica de fora deste diagrama, de propósito

- **Particionamento/expurgo da tabela** — necessário em paralelo (nota no
  card do Legado), mas é faxina de banco de dados, não um componente de
  evento. Sem isso, mesmo os Consumers em paralelo continuam disputando uma
  tabela lenta.
- **Por que CDC e não Outbox** — o legado já marca internamente quando uma
  fatura fica pronta; o CDC observa essa mudança de status que já existe, sem
  precisar de nenhuma alteração de código. Outbox fica como alternativa se a
  granularidade do CDC se mostrar barulhenta na prática.
