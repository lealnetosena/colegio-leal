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

## Posts da série

_(rascunhos completos em `posts/`, escritos e aguardando revisão antes de
publicar — link do post no LinkedIn e do artigo no Dev.to entram aqui quando
saírem)_

| # | Assunto | Post | Artigo |
|---|---|---|---|
| 1 | O problema + arquitetura proposta | [`01-linkedin.md`](posts/01-linkedin.md) | [`01-artigo.md`](posts/01-artigo.md) |
| 2 | CDC + processamento em paralelo | [`02-linkedin.md`](posts/02-linkedin.md) | [`02-artigo.md`](posts/02-artigo.md) |
| 3 | Finalização proativa + fan-out | [`03-linkedin.md`](posts/03-linkedin.md) | [`03-artigo.md`](posts/03-artigo.md) |
| 4 | Aquisição + Anti-Corruption Layer + contrato | [`04-linkedin.md`](posts/04-linkedin.md) | [`04-artigo.md`](posts/04-artigo.md) |
| 5 | Retry | [`05-linkedin.md`](posts/05-linkedin.md) | [`05-artigo.md`](posts/05-artigo.md) |
| 6 | Dead-letter | [`06-linkedin.md`](posts/06-linkedin.md) | [`06-artigo.md`](posts/06-artigo.md) |
| 7 | Observabilidade | [`07-linkedin.md`](posts/07-linkedin.md) | [`07-artigo.md`](posts/07-artigo.md) |
| 8 | Bancos: relacional + MongoDB | [`08-linkedin.md`](posts/08-linkedin.md) | [`08-artigo.md`](posts/08-artigo.md) |

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
