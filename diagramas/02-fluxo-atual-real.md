# Fluxo atual (como é hoje) — versão mais realista

> Rascunho de referência pra você redesenhar no draw.io. As premissas abaixo são
> do case (fictício): ajuste conforme a realidade que você viu, sem citar nomes.

## Os 4 problemas escolhidos (P1 a P4)
| # | Problema do aluno / da faculdade | Benefício da EDA que resolve |
|---|---|---|
| P1 | O faturamento não fecha em 1 dia | Escala e paralelismo |
| P2 | A nota fiscal (prefeitura) trava todo o resto | Isolamento de falha |
| P3 | Paguei o PIX e continua "em aberto" | Reação em tempo real (evento em vez de lote) |
| P4 | Ajustaram minha matrícula, mas boleto, nota e app seguem velhos | Propagação de mudança |

## 1. Visão estrutural — quem fala com quem

```mermaid
flowchart LR
    subgraph OPS["Operação"]
        SEC["Secretaria / Financeiro<br/>ajustes manuais"]
    end

    subgraph LEG["Sistema legado (monolito)"]
        ACAD["Acadêmico<br/>matérias, dependências,<br/>serviços solicitados"]
        CALC["Cálculo da mensalidade"]
        JOB["Job de faturamento<br/>aluno por aluno, em sequência"]
        CONC["Job de conciliação<br/>lê retornos em lote"]
        DB[("Banco relacional")]
    end

    subgraph EXT["Terceiros (fora do nosso controle)"]
        BANCO["Banco<br/>registro de boleto e retorno"]
        PSP["PSP<br/>PIX"]
        CARD["Gateway de cartão"]
        PREF["Prefeitura<br/>NFS-e"]
        PUSH["Push"]
        WA["WhatsApp Business"]
        TG["Telegram Bot API"]
    end

    subgraph CH["O que o aluno usa"]
        PORTAL["Portal (site)"]
        APP["App"]
        BOT["Chatbot"]
    end

    SEC -->|"trancamento, cancelamento de taxa"| ACAD
    ACAD --> CALC
    CALC --> JOB
    JOB <--> DB

    JOB -->|"1"| BANCO
    JOB -->|"2"| PSP
    JOB -->|"3"| CARD
    JOB -->|"4"| PREF
    JOB -->|"5"| PUSH
    JOB -->|"6"| WA
    JOB -->|"7"| TG

    BANCO -.->|"retorno em lote"| CONC
    PSP -.->|"consulta periódica"| CONC
    CONC -->|"dá baixa"| DB

    PORTAL -->|"lê direto"| DB
    APP --> PORTAL
    BOT --> PORTAL
    PUSH --> APP
    WA --> BOT
    TG --> BOT

    P1{{"P1 · Não fecha em 1 dia"}}
    P2{{"P2 · Nota fiscal trava o resto"}}
    P3{{"P3 · PIX pago, ainda em aberto"}}
    P4{{"P4 · Correção não se espalha"}}
    P1 -.-> JOB
    P2 -.-> PREF
    P3 -.-> CONC
    P4 -.-> SEC

    classDef prob fill:#ffe3e3,stroke:#d33,color:#900
    class P1,P2,P3,P4 prob
```

As setas 1 a 7 são chamadas **síncronas, em sequência**, feitas pelo job, aluno
por aluno.

## 2. Visão temporal — a virada de mês (P1 e P2)

```mermaid
sequenceDiagram
    autonumber
    participant J as Job de faturamento
    participant B as Banco (boleto)
    participant X as PSP (PIX)
    participant C as Gateway (cartão)
    participant P as Prefeitura (NFS-e)
    participant N as Avisos (push, WhatsApp, Telegram)

    loop Para cada um dos 3.000 alunos
        J->>J: calcula a mensalidade (matérias, dependências, taxas)
        J->>B: registra o boleto
        B-->>J: ok
        J->>X: cria a cobrança PIX
        X-->>J: QR Code
        J->>C: cria o link de cartão
        C-->>J: ok
        J->>P: envia a nota (RPS)
        P-->>J: protocolo
        loop até a prefeitura responder
            J->>P: consulta o protocolo
        end
        Note over J,P: prefeitura lenta ou fora do ar:<br/>o job inteiro fica esperando (P2)
        P-->>J: nota emitida
        J->>N: envia os avisos
        N-->>J: ok
    end
    Note over J: 3.000 alunos × cerca de 20 s ≈ 16h40 (P1)
```

## 3. Visão do dia a dia (P3 e P4)

**P3 — PIX pago, sistema "em aberto"**

```mermaid
sequenceDiagram
    participant A as Aluno
    participant X as PSP / Banco
    participant K as Job de conciliação
    participant L as Legado (banco de dados)
    participant V as Portal / App / Chatbot

    A->>X: paga o PIX às 10h02
    Note over X,K: o banco sabe na hora,<br/>mas o legado só descobre no próximo lote
    V->>L: consulta a situação
    L-->>V: em aberto (P3)
    K->>X: consulta os pagamentos (a cada poucas horas)
    X-->>K: pagamentos confirmados
    K->>L: dá baixa
```

**P4 — ajuste manual que não se espalha**

```mermaid
sequenceDiagram
    participant S as Secretaria
    participant L as Legado
    participant B as Banco (boleto)
    participant P as Prefeitura (nota)
    participant V as App / Chatbot

    S->>L: tranca a matéria depois do fechamento
    L->>L: recalcula o valor
    Note over L,P: nada avisa o boleto, a nota nem os canais (P4)
    B-->>V: boleto segue com o valor antigo
    P-->>V: nota segue com o valor antigo
    Note over S,V: alguém precisa cancelar e reemitir na mão
```

## Premissas do case (ajuste conforme a realidade que você viu)
- Cobrança e nota fiscal por aluno, todo mês.
- Cerca de 20 s por aluno somando todas as chamadas externas (boleto, PIX, cartão,
  prefeitura, avisos) — número fictício, serve pra conta do P1.
- A conciliação de pagamentos roda em lote, a cada poucas horas, a partir do
  retorno do banco e da consulta ao PSP.
- O portal lê direto do banco do legado; app e chatbot consultam o portal.
- Ajuste manual da secretaria não dispara nenhuma ação automática nos terceiros.
