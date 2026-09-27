# Visão de negócio — situação atual (sem jargão técnico)

> Este diagrama é pra quem não é dev entender o problema: gestor, product
> owner, recrutador. Zero menção a Kafka, banco de dados, API. O desenho
> técnico equivalente é `01-arquitetura-atual.md`.

```mermaid
flowchart LR
    A["Secretaria<br/>fecha o cálculo do mês"]
    B["Sistema<br/>gera a cobrança"]
    P["Aluno<br/>paga"]
    C["Banco / meios de pagamento<br/>confirma o pagamento"]
    F["Sistema<br/>emite a nota fiscal"]
    D["Prefeitura<br/>emite a nota"]
    G["Sistema<br/>avisa o aluno"]
    R["Aluno<br/>recebe boleto, nota e aviso"]

    A --> B --> P --> C --> F --> G --> R
    F -.->|"aciona a prefeitura<br/>(às vezes demora dias)"| D

    PB1{{"Não fecha em 1 dia"}}
    PB2{{"Nota demora a sair"}}
    PB3{{"Pagou, mas o sistema<br/>ainda mostra em aberto"}}
    PB4{{"Ajuste não reflete em<br/>nada que o aluno vê"}}
    PB1 -.-> B
    PB2 -.-> F
    PB3 -.-> C
    PB4 -.-> A

    classDef prob fill:#ffe3e3,stroke:#d33,color:#900
    class PB1,PB2,PB3,PB4 prob
    classDef ext fill:#fff3d6,stroke:#c90,color:#630
    class C,D ext
```

## Leitura
- 4 papéis, nenhum termo técnico: **Secretaria/Financeiro** (fecha o cálculo),
  **Sistema da faculdade** (processa), **Parceiros externos** (banco e
  prefeitura, fora do controle da faculdade), **Aluno** (paga e recebe).
- Os 4 problemas (PB1–PB4) aparecem nos mesmos pontos do desenho técnico —
  só que descritos em linguagem de negócio, não de sistema.
- Esse é o desenho pra abrir o artigo/post: qualquer leitor entende sem saber
  o que é fila, evento ou consumer.
