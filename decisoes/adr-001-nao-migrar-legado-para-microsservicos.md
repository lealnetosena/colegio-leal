# ADR-001 — Não migrar o legado para microsserviços; extrair por eventos ao redor dele

**Status:** Aceita (fictícia, parte do case)

## Contexto
O sistema acadêmico e financeiro da Faculdade Leal é um monolito com banco
relacional. Ele calcula mensalidade a partir de matérias cursadas,
dependências e taxas de serviço — regra de negócio testada e estável. Os
problemas reais (P1–P4) não estão nessa regra: estão em como o resultado do
cálculo é **entregue** a sistemas de terceiros (banco, PSP, prefeitura) e aos
canais do aluno (site, app, WhatsApp, Telegram), tudo de forma síncrona e em
cadeia.

## Decisão
Não reescrever o legado em microsserviços. Ele continua monolito, dono do
cálculo. As capacidades que causam os problemas — emissão de nota fiscal,
confirmação de pagamento, notificação por canal — são extraídas como serviços
independentes, que reagem a um evento (`mensalidade.gerada`) publicado pelo
legado. Padrão conhecido como *strangler fig*: o novo cresce em volta do
antigo, sem substituí-lo de uma vez.

## Alternativas consideradas
- **Reescrever tudo em microsserviços.** Rejeitada: custo e risco altos para
  reescrever uma regra de cálculo que já funciona; não ataca a causa raiz dos
  problemas (que é o acoplamento síncrono, não o monolito em si).
- **Manter tudo síncrono e só otimizar performance** (paralelizar threads,
  aumentar timeout). Rejeitada: não resolve isolamento de falha (P2) nem
  reação em tempo real (P3); só adia o problema.

## Consequências
**Positivas:** menor risco (o cálculo não é tocado), entrega incremental
(um consumer por vez, como a série mostra), cada capacidade nova escala
sozinha.

**Negativas (o preço de virar distribuído — resto da série cobre cada uma):**
- Entrega deixa de ser garantida na hora → é preciso lidar com falha parcial
  (post 5, retry) e mensagem que nunca dá certo (post 6, dead-letter).
- Rastrear "o que aconteceu com o aluno X" fica mais difícil → observabilidade
  vira necessidade, não luxo (post 7).
- Convivem, por um tempo, dois modelos: o legado ainda tem lógica interna
  síncrona; só a borda dele passa a publicar eventos.
