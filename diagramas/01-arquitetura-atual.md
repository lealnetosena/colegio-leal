# Arquitetura atual (síncrona) — Colégio Leal

Contexto: 3.000 alunos matriculados, mensalidade vencendo todo dia 5.

## Visão estrutural — quem fala com quem

```mermaid
flowchart LR
    subgraph Legado["Sistema de Gestão Escolar (legado)"]
        Job["Job de Faturamento<br/>(processa aluno por aluno, em sequência)"]
    end

    Job -->|"1 síncrono"| Pag[Gateway de Pagamento]
    Job -->|"2 síncrono"| NFSe[Prefeitura: emissão de NFS-e]
    Job -->|"3 síncrono"| Site[Portal do Site]
    Job -->|"4 síncrono"| App[App do Colégio]
    Job -->|"5 síncrono"| WA[WhatsApp Business API]
    Job -->|"6 síncrono"| TG[Telegram Bot API]

    style Job fill:#f96,stroke:#333
```

## Visão temporal — por que trava

```mermaid
sequenceDiagram
    participant Job as Job de Faturamento
    participant Pag as Gateway de Pagamento
    participant NFSe as Prefeitura (NFS-e)
    participant Site as Site
    participant App as App
    participant WA as WhatsApp
    participant TG as Telegram

    loop Para cada um dos 3.000 alunos
        Job->>Pag: gera boleto
        Pag-->>Job: ok
        Job->>NFSe: emite nota fiscal (NFS-e)
        NFSe-->>Job: nota emitida (PDF)
        Job->>Site: publica notificação
        Site-->>Job: ok
        Job->>App: envia push
        App-->>Job: ok
        Job->>WA: envia mensagem
        Note over Job,WA: se o WhatsApp demorar,<br/>TUDO nesse aluno espera
        WA-->>Job: ok (ou timeout)
        Job->>TG: envia mensagem
        TG-->>Job: ok
    end
```

## Problemas que essa arquitetura expõe

- **Cadeia bloqueante**: 6 chamadas síncronas em sequência, por aluno, multiplicadas por 3.000.
- **Acoplamento forte**: o job de faturamento (legado) precisa conhecer a API de todo mundo. Adicionar um canal novo significa mexer no código legado.
- **Sem isolamento de falha**: se o WhatsApp cai, os alunos que ainda não foram processados ficam sem NADA (nem site, nem app, nem Telegram), porque a cadeia trava.
- **Janela de tempo em risco**: o job precisa terminar antes de um horário (ex.: antes das 6h). Qualquer lentidão externa ameaça essa janela.

_Status: rascunho v1, gerado em colaboração com o Claude em 2026-09-26 — ajustar números/labels à vontade._
