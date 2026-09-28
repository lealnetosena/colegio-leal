# Faculdade Leal — case de arquitetura orientada a eventos

Case fictício, construído em público, sobre como um sistema legado que não
escala — 4 dias pra fechar o faturamento, site quebrando sob carga — é
escalado com arquitetura orientada a eventos **sem ser reescrito**.

Stack: Apache Camel, Kafka (self-hosted, via Docker), CDC (Debezium),
MongoDB, banco relacional.

## Os problemas (ver `diagramas/01-arquitetura-atual.md`)

- **P1** — o faturamento não fecha em 1 dia (tabela sem expurgo + processamento
  serial/RBAR)
- **P2** — boleto/PIX sob demanda quebra com carga (mesma tabela, procedure
  de finalização acionada a cada clique)
- **P3** — só existe o canal site; app e WhatsApp são roadmap
- **P4** — aquisição de outra faculdade, com sistema legado próprio

## A arquitetura proposta (ver `diagramas/02-arquitetura-proposta.md`)

6 movimentos, nenhum exige reescrever o legado: Strangler Fig → CDC → 
Competing Consumers → particionamento/purge (em paralelo, fora do EDA) →
CQRS + Materialized View → Publisher-Subscriber (fan-out) →
Anti-Corruption Layer (aquisição).

## Diagramas

Fontes em [`diagramas/`](diagramas):
- `01-arquitetura-atual.md` — estado atual (o problema)
- `02-arquitetura-proposta.md` — estado alvo (os 6 movimentos)
- `03-post2-calculo-paralelo.md`, `04-post3-fanout-leitura.md`,
  `05-post6-dead-letter.md` — fatias incrementais, usadas nos posts
  correspondentes

## Repositórios de código

| Repositório | Papel |
|---|---|
| [`legado-faculdade-leal`](https://github.com/lealnetosena/legado-faculdade-leal) | Sistema legado (Aplicação + Procedures + Tabela) |
| [`consumer-fiscal-pdf`](https://github.com/lealnetosena/consumer-fiscal-pdf) | Existe, mas nome/escopo é de antes do reset de 27/09 — revisar contra a arquitetura proposta atual antes de implementar |
| [`consumer-notif-site`](https://github.com/lealnetosena/consumer-notif-site), [`-app`](https://github.com/lealnetosena/consumer-notif-app), [`-whatsapp`](https://github.com/lealnetosena/consumer-notif-whatsapp), [`-telegram`](https://github.com/lealnetosena/consumer-notif-telegram) | Idem — nomes/escopo de antes do reset |

> Nenhum repositório de código foi criado/renomeado para os componentes
> novos da arquitetura proposta (CDC, Consumers de Cálculo/Finalização,
> Materialized View, Anti-Corruption Layer). Isso fica para quando a
> implementação real começar — os posts, por enquanto, são só texto e
> diagrama.
