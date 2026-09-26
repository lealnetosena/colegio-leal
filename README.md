# Colégio Leal — case de arquitetura orientada a eventos

Case fictício, construído em público, sobre como uma rede de escolas resolve o
problema de faturar a mensalidade de milhares de alunos sem travar o sistema
inteiro toda vez que uma área terceira demora a responder.

Stack: Apache Camel, Kafka (self-hosted, via Docker), MongoDB, banco relacional.

## Repositórios do case

| Repositório | Papel |
|---|---|
| [`colegio-leal`](.) | Este repo: visão geral, diagramas |
| [`legado-colegio-leal`](../legado-colegio-leal) | Sistema legado (matrícula, mensalidade, monolito + relacional) |
| [`consumer-fiscal-pdf`](../consumer-fiscal-pdf) | Gera o comprovante em PDF para declaração de IR |
| [`consumer-notif-site`](../consumer-notif-site) | Notificação: portal do site |
| [`consumer-notif-app`](../consumer-notif-app) | Notificação: push no app |
| [`consumer-notif-whatsapp`](../consumer-notif-whatsapp) | Notificação: bot de WhatsApp |
| [`consumer-notif-telegram`](../consumer-notif-telegram) | Notificação: bot de Telegram |

> URLs reais do GitHub entram aqui assim que os repositórios forem publicados.

## Posts da série

_(preencher conforme os posts forem saindo — link do post no LinkedIn + link do artigo no Dev.to)_

1. O problema + arquitetura atual — _a publicar_
2. Implementação básica: evento `mensalidade.gerada` — _a publicar_
3. Fan-out: site, app, WhatsApp, Telegram — _a publicar_
4. Quando o contrato quebra (e a rede vira rede de verdade) — _a publicar_
5. Retry — _a publicar_
6. Dead-letter — _a publicar_
7. Observabilidade — _a publicar_
8. Bancos: relacional + MongoDB — _a publicar_

## Diagramas

Fontes em [`diagramas/`](diagramas), exportados em PNG/SVG pros artigos.
