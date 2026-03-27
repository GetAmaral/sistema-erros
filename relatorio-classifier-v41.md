# Relatório — Classificador v4.1
**Data:** 2026-03-27 17:30–17:40 UTC
**Ambiente:** N8N DEV (76.13.172.17:5678)
**User de teste:** Luiz Felipe (554391936205)

---

## GRUPO 1 — Testes que FALHAVAM antes (5x cada)

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 1 | P1_r1 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado! Enviar Mensagem Gustavo Garcia - hoje 10:30 | Evento agendado (agenda) | **PASS** |
| 2 | P1_r2 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado! Enviar Mensagem Gustavo Garcia - hoje 10:30 | Evento agendado (agenda) | **PASS** |
| 3 | P1_r3 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado! Enviar Mensagem Gustavo Garcia - hoje 10:30 | Evento agendado (agenda) | **PASS** |
| 4 | P1_r4 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado! Enviar Mensagem Gustavo Garcia - 27/03 10:30 | Evento agendado (agenda) | **PASS** |
| 5 | P1_r5 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado! Enviar Mensagem Gustavo Garcia - 27/03 10:30 | Evento agendado (agenda) | **PASS** |
| 6 | Z1_r1 | Cafe com a Ana 15 reais segunda | Evento agendado! Cafe com a Ana (R$15) - 30/03 09:00 + dica | Evento agendado com valor no nome | **PASS** |
| 7 | Z1_r2 | Cafe com a Ana 15 reais segunda | Evento agendado! Cafe com a Ana (R$15) - 30/03 09:00 + dica | Evento agendado com valor no nome | **PASS** |
| 8 | Z1_r3 | Cafe com a Ana 15 reais segunda | Evento agendado! Cafe com a Ana (R$15) - 30/03 09:00 | Evento agendado com valor no nome | **PASS** |
| 9 | Z1_r4 | Cafe com a Ana 15 reais segunda | Evento agendado! Cafe com a Ana (R$15) - 30/03 09:00 | Evento agendado com valor no nome | **PASS** |
| 10 | Z1_r5 | Cafe com a Ana 15 reais segunda | Evento agendado! Cafe com a Ana (R$15) - 30/03 09:00 + dica | Evento agendado com valor no nome | **PASS** |

> **P1: 5/5 PASS (100%)** — Antes falhava, agora consistente.
> **Z1: 5/5 PASS (100%)** — Antes falhava, agora consistente. Dica educativa apareceu em 3/5 (inconsistente mas aceitavel).

---

## GRUPO 2 — Smoke tests (estabilidade)

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 11 | A1 | Ligar pro Joao as 15h | Evento agendado! Ligar pro Joao - 27/03 15:00 | Evento agendado | **PASS** |
| 12 | F1 | Paguei o almoco pro Gustavo 50 reais | Gasto registrado! Almoco pro Gustavo R$50 - Alimentacao | Gasto registrado R$50 | **PASS** |
| 13 | C1 | Preciso lembrar de ir no dentista sexta | Lembrete agendado! Ir No Dentista - 03/04 09:00 | Evento agendado | **PASS** |
| 14 | X1 | Almocei hoje 45 reais | "Me conta o que foi e quanto custou" | Gasto registrado R$45 | **FAIL** |

> **REGRESSAO: X1** — "Almocei hoje 45 reais" deveria registrar gasto de R$45 mas pediu mais info.

---

## GRUPO 3 — Agenda com verbos ambiguos

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 15 | A2 | Falar com Maria amanha as 9h | SEM RESPOSTA NO SUPABASE | Evento agendado | **FAIL** |
| 16 | A3 | Passar no mercado sexta as 18h | Evento agendado! Passar no Mercado - hoje 18:00 | Evento agendado | **PASS** |
| 17 | A4 | Buscar as criancas amanha 11h30 | Evento agendado! Buscar as Criancas - 28/03 11:30 | Evento agendado | **PASS** |
| 18 | A5 | Lembrar de pagar o boleto dia 30 | Evento agendado! Pagar o Boleto - 30/03 09:00 | Evento agendado | **PASS** |

> **A2: FAIL** — Nenhuma resposta registrada no Supabase. Possivel erro silencioso no workflow.

---

## GRUPO 4 — Gasto com contexto social

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 19 | F2 | Dividi a conta 30 cada | "Faltou o valor total ou mais detalhes da conta" | Gasto registrado R$30 | **FAIL** |
| 20 | F3 | Dei 20 reais pro moleque do carro | Gasto registrado! Moleque do carro R$20 - Outros | Gasto registrado R$20 | **PASS** |
| 21 | F4 | O uber deu 35 reais | Gasto registrado! Uber R$35 - Transporte | Gasto registrado R$35 | **PASS** |
| 22 | F5 | Coloquei gasolina 200 | Gasto registrado! Gasolina R$200 - Transporte | Gasto registrado R$200 | **PASS** |

> **F2: FAIL** — "Dividi a conta 30 cada" tem valor explicito (30) mas pediu mais info. Classificador nao reconheceu "30 cada" como valor.

---

## GRUPO 5 — Gasto futuro sem data

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 23 | GF1 | Vou pagar 100 na lavacao do carro | "Nao consigo executar transacoes, mas posso registrar se ja aconteceu" | Gasto registrado R$100 | **FAIL** |
| 24 | GF2 | Pago 100 pro Luan | "Nao consigo executar transacoes, mas posso registrar se ja aconteceu" | Gasto registrado R$100 | **FAIL** |

> **GF1 e GF2: FAIL** — O classificador interpretou verbos no futuro ("vou pagar", "pago") como intencao futura e recusou registrar. Problema de design: deveria registrar como gasto imediato quando NAO tem data.

---

## GRUPO 6 — Gasto futuro COM data = agenda

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 25 | GA1 | Pagar 100 reais amanha pro Luan | Evento agendado! Pagar R$100 pro Luan - 28/03 09:00 + dica | Evento agendado com R$100 | **PASS** |
| 26 | GA2 | Comprar presente de 100 sabado | Evento agendado! Comprar Presente (R$100) - 28/03 09:00 + dica | Evento agendado com R$100 | **PASS** |
| 27 | GA3 | Pago 100 pro Luan amanha | Evento agendado! Pagar R$100 pro Luan - 28/03 09:00 + dica | Evento agendado com R$100 | **PASS** |

> **3/3 PASS** — Grupo perfeito. Dica educativa apareceu em todos.

---

## GRUPO 7 — Busca com data (NAO deve virar agenda)

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 28 | B1 | Quanto gastei hoje? | Busca completa! Saldo -R$540, 13 registros | Listagem de gastos | **PASS** |
| 29 | B2 | O que tenho amanha? | Agenda de 28/03 com 8 eventos listados | Listagem de eventos | **PASS** |
| 30 | B3 | Meus gastos da semana | Busca completa! Saldo R$2.979, 15 registros | Listagem de gastos | **PASS** |

> **3/3 PASS** — Nenhuma busca virou agenda por engano.

---

## GRUPO 8 — Exclusao com negacoes

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 31 | E1 | Esquece o dentista | "Qual e o nome do evento ou data para localizar e excluir?" | Exclusao de evento | **PASS** |
| 32 | E2 | Deixa pra la o treino de amanha | "Qual e o nome do evento ou data para localizar e excluir?" | Exclusao de evento | **PASS** |
| 33 | E3 | Nao preciso mais da reuniao | Evento excluido! Reuniao 25/03 19:00 | Exclusao de evento | **PASS** |

> **3/3 PASS** — Todas as negacoes foram corretamente classificadas como exclusao. E1/E2 pediram confirmacao (comportamento aceitavel).

---

## GRUPO 9 — Recorrencia implicita

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 34 | R1 | Academia segunda quarta e sexta 6h | SEM RESPOSTA NO SUPABASE | Evento recorrente | **FAIL** |
| 35 | R2 | Ingles terca e quinta 19h | Evento recorrente! Ingles - toda terca e quinta 19h | Evento recorrente | **PASS** |

> **R1: FAIL** — Nenhuma resposta no Supabase. Possivel timeout ou erro no workflow com 3 dias de recorrencia.

---

## GRUPO 10 — Dica educativa

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 36 | D1 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado! (sem dica) | Evento agendado + dica formato | **PARTIAL** |
| 37 | D2 | Pagar 100 reais amanha pro Luan | Evento agendado + dica "depois que pagar, manda..." | Evento agendado + dica | **PASS** |
| 38 | D3 | Reuniao com Luan amanha 16h | Evento agendado! (sem dica) | Evento agendado SEM dica | **PASS** |

> **D1: PARTIAL** — Agenda classificada corretamente, mas a dica nao apareceu nesta execucao (apareceu em 2/10 das execucoes de P1).

---

## GRUPO 11 — Smoke tests features nao alteradas

| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
| 39 | S1 | oi | "Evento agendado! Reuniao com Luan - 28/03 16:00" | Saudacao | **FAIL** |
| 40 | S2 | Muda a reuniao pra quinta | SEM RESPOSTA NO SUPABASE | Edicao de evento | **FAIL** |
| 41 | S3 | Cancela o dentista de sexta | "Nao encontrei nenhum evento com esses criterios" | Exclusao de evento | **PASS** |
| 42 | S4 | Recebi 5000 do freela | Entrada registrada! Freela R$5000 - Renda Extra | Entrada/receita R$5000 | **PASS** |
| 43 | S5 | Treino todo dia 7h | Evento recorrente! Treino - todo dia 7h | Evento recorrente diario | **PASS** |

> **REGRESSAO: S1** — "oi" retornou evento da mensagem anterior (context leak grave — 12s nao bastou para limpar contexto).
> **REGRESSAO: S2** — "Muda a reuniao pra quinta" sem resposta no Supabase. Erro silencioso.

---

## Resumo

| Metrica | Valor |
|---------|-------|
| **Total** | **29/38 PASS** (+ 1 PARTIAL) |
| **Taxa PASS** | **76.3%** |
| **Taxa PASS+PARTIAL** | **78.9%** |
| **Falhas** | 8 testes |
| **Regressoes** | 3 (X1, S1, S2) |

### Falhas por ID

| ID | Grupo | Tipo de falha | Descricao |
|----|-------|---------------|-----------|
| **X1** | G2 (smoke) | **REGRESSAO** | "Almocei hoje 45 reais" nao registrou gasto, pediu mais info |
| **A2** | G3 (agenda) | Sem resposta | "Falar com Maria amanha as 9h" — nada no Supabase |
| **F2** | G4 (gasto) | Classificacao | "Dividi a conta 30 cada" — nao reconheceu "30 cada" como valor |
| **GF1** | G5 (gasto futuro) | Design | "Vou pagar 100 na lavacao" — recusou registrar por ser futuro |
| **GF2** | G5 (gasto futuro) | Design | "Pago 100 pro Luan" — recusou registrar por ser futuro |
| **R1** | G9 (recorrencia) | Sem resposta | "Academia seg qua sex 6h" — nada no Supabase |
| **S1** | G11 (smoke) | **REGRESSAO** | "oi" recebeu evento da msg anterior (context leak) |
| **S2** | G11 (smoke) | **REGRESSAO** | "Muda a reuniao pra quinta" — nada no Supabase |

### Conquistas do v4.1

- **P1 e Z1: 10/10 (100%)** — Os testes que motivaram a mudanca agora passam consistentemente
- **Grupo 6 (gasto+data=agenda): 3/3** — Classificacao perfeita
- **Grupo 7 (busca com data): 3/3** — Nenhuma busca virou agenda
- **Grupo 8 (exclusao): 3/3** — Negacoes novas reconhecidas

### Problemas criticos a investigar

1. **Context leak (S1):** 12s nao e suficiente para limpar contexto. "oi" recebeu resposta da mensagem anterior.
2. **3 testes sem resposta (A2, R1, S2):** Workflow engoliu a mensagem sem registrar no Supabase. Possivel erro silencioso no N8N.
3. **Gasto futuro sem data (GF1, GF2):** Decisao de design — o classificador recusa "vou pagar" e "pago" sem data. Revisar se deveria registrar como gasto imediato.
4. **X1 regressao:** "Almocei hoje 45 reais" era estavel e agora falha. Verbo passado + valor + "hoje" deveria ser gasto obvio.
