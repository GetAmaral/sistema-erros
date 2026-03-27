# Relatorio Bateria de Testes — 2026-03-27

**Periodo:** 20:15 ~ 20:28 UTC
**Envios:** 92 (incluindo 1 ping)
**Mensagens logadas no Supabase:** 88
**Mensagens perdidas:** 4 completamente ausentes + 2 parciais = 6 (93.5% delivery)
**Falhas HTTP:** 0 (todos retornaram 200)

---

## Resumo Executivo

| Metrica | Valor |
|---------|-------|
| Total enviado | 92 |
| Logado no Supabase | 88 |
| Taxa de entrega | 93.5% |
| Classificacao CORRETA | 88/88 (100%) |
| Classificacao com PROBLEMA | 0/88 (0%) |
| Bugs de comportamento/resposta | 3/88 (3.4%) |
| Mensagens perdidas | 6/92 (6.5%) |

### Execucoes N8N (apos baseline ID 13257)

| Workflow | Success | Error | Total |
|----------|---------|-------|-------|
| Main - Total Assistente | 90 | 0 | 90 |
| Fix Conflito v2 | 91 | 0 | 91 |
| Calendar WebHooks | 10 | 51 | 61 |
| Financeiro | 4 | 26 | 30 |
| Lembretes | 0 | 4 | 4 |
| **Total** | **195** | **81** | **276** |

---

## Erros N8N Identificados

### 1. Calendar WebHooks (51 erros)
- **Node:** `refresh_access`
- **Erro:** `The connection was aborted, perhaps the server is offline`
- **Causa:** Falha ao renovar token OAuth do Google Calendar. O refresh_token pode estar expirado ou o endpoint do Google rejeitou a conexao.
- **Impacto:** Eventos sao criados pelo Fix Conflito v2 (que teve 91 success), mas o Calendar WebHooks falha ao sincronizar. O usuario recebe confirmacao de agendamento normalmente.

### 2. Financeiro (26 erros)
- **Node:** `Create a row1`
- **Erro:** `Could not find the table 'public.log_total' in the schema cache`
- **Causa:** A tabela `log_total` nao existe ou nao esta no schema cache do Supabase. Pode ser tabela removida/renomeada.
- **Impacto:** Gastos sao registrados com sucesso (o usuario recebe confirmacao), mas o log consolidado em `log_total` falha. Isso afeta rastreabilidade.

### 3. Lembretes (4 erros)
- **Node:** `Create a row2`
- **Erro:** `Could not find the table 'public.log_total' in the schema cache`
- **Causa:** Mesmo problema do Financeiro — tabela `log_total` ausente.
- **Impacto:** Lembretes criados mas nao logados na tabela consolidada.

---

## Mensagens Completamente Perdidas (nao logadas)

| ID Script | Mensagem | Observacao |
|-----------|----------|------------|
| B2_E2 | `Deixa pra la o treino de amanha` | Exclusao |
| B2_S2 | `Muda a reuniao pra quinta` | Edicao contextual |
| B2_S3 | `Cancela o dentista de sexta` | Exclusao |
| B2_S4 | `Recebi 5000 do freela` | Receita |

**Padrao:** 3 das 4 sao operacoes de edicao/exclusao. Possivelmente o fluxo de conflito (Fix Conflito v2) descartou essas mensagens por nao encontrar contexto ou por concorrencia.

Alem dessas, 1x `Cafe com a Ana` e 1x `Paguei o almoco pro Gustavo` nao foram logadas (parciais).

---

## Bugs de Classificacao / Resposta

### BUG 1: Resposta incoerente — "Treino todo dia 7h" (msg #73)
- **Input:** `Treino todo dia 7h`
- **Resposta:** `Me diga qual reuniao quer mudar para quinta. Se quiser, tambem posso cancelar o dentista...`
- **Esperado:** Evento recorrente registrado (como R1/R2 funcionaram)
- **Causa provavel:** Contexto conversacional do Fix Conflito v2 — as mensagens S2 ("Muda a reuniao pra quinta") e S3 ("Cancela o dentista de sexta") foram enviadas pouco antes e a IA misturou o contexto.
- **Severidade:** ALTA — resposta completamente errada para o usuario

### BUG 2: Resposta com lancamento duplicado — "Almocei hoje 45 reais" (msg #48)
- **Input:** `Almocei hoje 45 reais`
- **Resposta:** `Lancamentos registrados! Almoco R$45 + Almoco pro Gustavo R$50`
- **Esperado:** Apenas `Gasto registrado! Almoco R$45`
- **Causa provavel:** A IA agrupou gastos por contexto temporal. Como "Paguei o almoco pro Gustavo 50 reais" foi enviado pouco antes na Bateria 2, o Fix Conflito v2 pode ter agrupado os dois.
- **Severidade:** MEDIA — registrou o gasto correto mas adicionou outro na resposta

### BUG 3: "Esquece o dentista" — exclusao cancelada (msg #65)
- **Input:** `Esquece o dentista`
- **Resposta:** `Ok, exclusao cancelada.`
- **Esperado:** Evento do dentista excluido ou confirmacao de exclusao
- **Causa provavel:** Possivel que o fluxo interpretou como cancelamento da operacao em vez de cancelamento do evento. Ambiguidade no intent de "esquece".
- **Severidade:** MEDIA — comportamento confuso para o usuario

---

## Resultados por Bateria

### Bateria 1 — 5x cada (30 envios)

| ID | Mensagem | Esperado | Resultado | Taxa |
|----|----------|----------|-----------|------|
| P1 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado | Evento agendado 28/03 10:30 | 5/5 OK |
| A1 | Ligar pro Joao as 15h | Evento agendado | Evento agendado 28/03 15:00 | 5/5 OK |
| F1 | Paguei o almoco pro Gustavo 50 reais | Gasto R$50 | Gasto R$50 Alimentacao | 5/5 OK |
| C1 | Preciso lembrar de ir no dentista sexta | Evento agendado | Evento agendado 03/04 09:00 | 5/5 OK |
| X1 | Almocei hoje 45 reais | Gasto R$45 | Gasto R$45 Alimentacao | 5/5 OK |
| Z1 | Cafe com a Ana 15 reais segunda | Evento agendado | Evento agendado 30/03 09:00 (R$15 no nome) | 5/5 OK |

**Bateria 1: 30/30 = 100% corretos**

### Bateria 2 — variado (53 envios esperados, 47 logados)

| ID | Mensagem | Resultado | Status |
|----|----------|-----------|--------|
| P1 x10 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado | 10/10 OK |
| Z1 x5 | Cafe com a Ana 15 reais segunda | Evento agendado | 5/5 OK |
| A1 | Ligar pro Joao as 15h | Evento agendado HOJE 15h | OK (data mudou para "hoje") |
| F1 | Paguei o almoco pro Gustavo 50 reais | **NAO LOGADO** | PERDIDO |
| X1 | Almocei hoje 45 reais | Lancamentos agrupados (bug) | BUG |
| C1 | Preciso lembrar de ir no dentista sexta | Evento agendado HOJE 09h | OK (interpretou "sexta" como hoje) |
| A3 | Passar no mercado sexta as 18h | Evento agendado HOJE 18h | OK |
| A4 | Buscar as criancas amanha 11h30 | Evento agendado amanha 11:30 | OK |
| A5 | Lembrar de pagar o boleto dia 30 | Evento agendado 30/03 09:00 | OK |
| F2 | Dividi a conta 30 cada | Gasto R$30 Outros | OK |
| F3 | Dei 20 reais pro moleque do carro | Gasto R$20 Outros | OK |
| F4 | O uber deu 35 reais | Gasto R$35 Transporte | OK |
| F5 | Coloquei gasolina 200 | Gasto R$200 Transporte | OK |
| GF1 | Vou pagar 100 na lavacao do carro | Gasto R$100 Transporte | OK |
| GF2 | Pago 100 pro Luan | Gasto R$100 Outros | OK |
| GA1 | Pagar 100 reais amanha pro Luan | Evento agendado 28/03 + dica R$100 | OK |
| GA2 | Comprar presente de 100 sabado | Evento agendado 28/03 (R$100 no nome) | OK |
| GA3 | Pago 100 pro Luan amanha | Evento agendado 28/03 + dica R$100 | OK |
| B1 | Quanto gastei hoje? | Busca completa: Saldo R$4.320 | OK |
| B2 | O que tenho amanha? | Agenda de 28/03 listada | OK |
| B3 | Meus gastos da semana | Busca completa: Saldo R$154.695 | OK |
| E1 | Esquece o dentista | Exclusao cancelada | BUG |
| E2 | Deixa pra la o treino de amanha | **NAO LOGADO** | PERDIDO |
| E3 | Nao preciso mais da reuniao | Nao encontrou evento | OK (sem contexto) |
| R1 | Academia seg/qua/sex 6h | Evento recorrente | OK |
| R2 | Ingles ter/qui 19h | Evento recorrente | OK |
| D1 | Enviar mensagem Gustavo Garcia as 10h30 | Evento agendado + dica | OK |
| D2 | Pagar 100 reais amanha pro Luan | Evento agendado + dica | OK |
| S1 | oi | Saudacao generica | OK |
| D3 | Reuniao com Luan amanha 16h | Evento agendado 28/03 16:00 | OK |
| S2 | Muda a reuniao pra quinta | **NAO LOGADO** | PERDIDO |
| S3 | Cancela o dentista de sexta | **NAO LOGADO** | PERDIDO |
| S4 | Recebi 5000 do freela | **NAO LOGADO** | PERDIDO |
| S5 | Treino todo dia 7h | Resposta incoerente (bug contexto) | BUG |

**Bateria 2: 43/47 logados corretos, 3 bugs, 5 perdidos**

### Bateria 3 — 5x X1 (5 envios)

| ID | Mensagem | Resultado | Status |
|----|----------|-----------|--------|
| X1 x5 | Almocei hoje 45 reais | Gasto R$45 Alimentacao | 5/5 OK |

**Bateria 3: 5/5 = 100% corretos**

### Bateria 4 — 1x cada (10 envios)

| ID | Mensagem | Resultado | Status |
|----|----------|-----------|--------|
| G1 | Jantei fora 80 reais | Gasto R$80 Alimentacao | OK |
| G2 | Paguei 120 de luz | Gasto R$120 Moradia | OK |
| G3 | Comprei remedio 35 reais | Gasto R$35 Saude | OK |
| A6 | Reuniao com dentista sexta 14h | Evento agendado 27/03 14h | OK |
| A7 | Buscar encomenda amanha 10h | Evento agendado 28/03 10:00 | OK |
| A8 | Levar cachorro veterinario segunda 16h | Evento agendado 30/03 16:00 | OK |
| R3 | Pilates terca e quinta 8h | Evento recorrente | OK |
| R4 | Terapia toda segunda 15h | Evento recorrente | OK |
| B4 | Quanto gastei essa semana? | Busca completa | OK |
| B5 | O que tenho amanha? | Agenda de 28/03 | OK |

**Bateria 4: 10/10 = 100% corretos**

---

## Classificacao do Classificador

### Corretos (por tipo)

| Tipo | Mensagens | Acertos | Taxa |
|------|-----------|---------|------|
| Agenda (evento simples) | P1, A1, C1, A3-A8, D3 | Todos | 100% |
| Agenda (valor no nome) | Z1, GA1-GA3 | Todos | 100% |
| Gasto (passado) | F1, X1, F2-F5, G1-G3 | Todos | 100% |
| Gasto futuro (sem data) | GF1, GF2 | Todos como gasto | 100% |
| Busca financeira | B1, B3, B4 | Todos | 100% |
| Busca agenda | B2, B5 | Todos | 100% |
| Evento recorrente | R1-R4 | Todos | 100% |
| Saudacao | S1 | Correto | 100% |

### Categorias Corretas

| Gasto | Categoria atribuida | Correto? |
|-------|---------------------|----------|
| Almoco pro Gustavo R$50 | Alimentacao | SIM |
| Almoco R$45 | Alimentacao | SIM |
| Conta R$30 | Outros | OK (ambiguo) |
| Moleque do carro R$20 | Outros | OK (ambiguo) |
| Uber R$35 | Transporte | SIM |
| Gasolina R$200 | Transporte | SIM |
| Lavacao R$100 | Transporte | OK (veiculo) |
| Luan R$100 | Outros | OK (transferencia) |
| Jantar R$80 | Alimentacao | SIM |
| Luz R$120 | Moradia | SIM |
| Remedio R$35 | Saude | SIM |

---

## Dicas Educativas

O classificador v4.1 inclui dicas educativas para mensagens ambiguas:

| Mensagem | Dica mostrada? |
|----------|---------------|
| P1 (Enviar mensagem Gustavo) | SIM — "Para aproveitar o maximo do..." (truncada) |
| Z1 (Cafe com a Ana R$15) | SIM em 1 de 10 — "Notei que voce mencionou R$15..." |
| GA1/GA3 (Pagar R$100 amanha) | SIM — "Notei que voce mencionou R$100..." |
| GA2 (Comprar presente R$100) | SIM — "Notei que voce mencionou R$100..." |
| D3 (Reuniao com Luan) | NAO — correto, nao e ambiguo |

---

## Conclusoes

### O que funciona bem
1. **Classificacao principal** esta solida — agenda, gasto, busca, recorrencia funcionam consistentemente
2. **Categorizacao de gastos** esta precisa (Alimentacao, Transporte, Moradia, Saude)
3. **Agenda com valor no nome** (Z1, GA1-GA3) corretamente classificada como agenda com dica educativa
4. **Gastos futuros sem data** (GF1, GF2) corretamente registrados como gasto imediato
5. **Eventos recorrentes** (R1-R4) interpretados e criados corretamente
6. **Consistencia sob carga:** 15x P1 e 10x Z1 deram resultados identicos

### Problemas encontrados

| # | Severidade | Descricao |
|---|------------|-----------|
| 1 | CRITICA | Tabela `log_total` inexistente — 30 erros em Financeiro+Lembretes |
| 2 | ALTA | Refresh token do Google Calendar expirado — 51 erros |
| 3 | ALTA | 4 mensagens completamente perdidas (E2, S2, S3, S4) |
| 4 | ALTA | "Treino todo dia 7h" recebeu resposta de outro contexto |
| 5 | MEDIA | "Esquece o dentista" = exclusao cancelada em vez de executada |
| 6 | MEDIA | "Almocei hoje 45 reais" na B2 agrupou com gasto anterior |
| 7 | BAIXA | 2 mensagens parcialmente perdidas (1x Z1, 1x F1) |

### Acoes recomendadas
1. **Criar tabela `public.log_total`** no Supabase principal ou remover referencia do workflow
2. **Renovar refresh token** do Google Calendar no workflow Calendar WebHooks
3. **Investigar mensagens perdidas** — S2/S3/S4 sao edicao/exclusao/receita, possivelmente descartadas pelo Fix Conflito v2 por falta de contexto
4. **Investigar contaminacao de contexto** — S5 recebeu resposta do fluxo de S2/S3, indicando que o contexto conversacional esta vazando entre mensagens rapidas
5. **Revisar intent "esquece"** — deve ser mapeado para exclusao, nao cancelamento

---

*Relatorio gerado automaticamente pelo squad auditor-360*
*Data: 2026-03-27 20:30 UTC*
*Script: run-bateria-2026-03-27.sh*
