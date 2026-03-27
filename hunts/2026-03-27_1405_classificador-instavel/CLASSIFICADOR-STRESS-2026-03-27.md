# Relatório de Auditoria — Estresse do Classificador (Escolher Branch)

**Data:** 2026-03-27
**Auditor:** Lupa (Inspector)
**Ambiente:** DEV (http://76.13.172.17:5678)
**User de teste:** Luiz Felipe (554391936205)
**Metodologia:** Envio de 5 repetições por mensagem, com limpeza de contexto automática (~10s), para medir **consistência** do classificador.

---

## Objetivo

Verificar se o classificador (Escolher Branch) no Fix Conflito v2 produz resultados **consistentes** quando recebe a mesma mensagem repetidas vezes com contexto limpo. Foco em mensagens **linguisticamente ambíguas** — onde a intenção é clara para um humano, mas as palavras podem confundir o LLM classificador.

---

## Teste Piloto — "Enviar mensagem Gustavo Garcia as 10h30"

| # | Horário | Resposta da IA | Veredicto |
|---|---------|----------------|-----------|
| 1 | 14:05:38 | "Essa mensagem não parece ser sobre agenda." | FAIL |
| 2 | 14:05:58 | "Essa mensagem não parece ser sobre agenda." | FAIL |
| 3 | 14:06:18 | "Evento agendado! Enviar Mensagem a Gustavo Garcia hoje às 10h30" | PASS |
| 4 | 14:06:35 | "Essa mensagem não parece ser sobre agenda." | FAIL |
| 5 | 14:07:08 | "Evento agendado! Enviar Mensagem Gustavo Garcia hoje às 10:30" | PASS |

**Taxa: 2/5 (40%) — INSTAVEL**

---

## Bateria de Estresse — 5 Grupos

### Grupo A1 — Agenda com verbo ambíguo
**Mensagem:** `"Ligar pro João às 15h"`
**Armadilha:** "ligar" = verbo de ação, não soa como "agendar evento"
**Intenção esperada:** Agenda (criar evento)

| # | Horário | Resposta | Veredicto |
|---|---------|----------|-----------|
| 1 | 14:18:02 | Evento agendado! Ligar para João hoje às 15:00 | PASS |
| 2 | 14:18:11 | Evento agendado! Ligar para João hoje às 15:00 | PASS |
| 3 | 14:18:24 | Evento agendado! Ligar para João hoje às 15:00 | PASS |
| 4 | 14:18:36 | Evento agendado! Ligar para João hoje às 15:00 | PASS |
| 5 | 14:18:49 | Evento agendado! Ligar para João hoje às 15:00 | PASS |

**Taxa: 5/5 (100%) — ESTAVEL**
**Notas:** Classificador acertou 100%. O padrão "verbo + pessoa + horário" é forte o suficiente.

---

### Grupo F1 — Financeiro com contexto social
**Mensagem:** `"Paguei o almoço pro Gustavo 50 reais"`
**Armadilha:** "paguei pro Gustavo" parece interpessoal, não registro de gasto
**Intenção esperada:** Financeiro (registrar gasto)

| # | Horário | Resposta | Veredicto |
|---|---------|----------|-----------|
| 1 | 14:19:02 | Gasto registrado! Almoço pro Gustavo R$50 Alimentação | PASS |
| 2 | 14:19:14 | Gasto registrado! Almoço pro Gustavo R$50 Alimentação | PASS |
| 3 | 14:19:25 | Gasto registrado! Almoço pro Gustavo R$50 Alimentação | PASS |
| 4 | 14:19:38 | Gasto registrado! Almoço pro Gustavo R$50 Alimentação | PASS |
| 5 | 14:19:50 | Gasto registrado! Almoço pro Gustavo R$50 Alimentação | PASS |

**Taxa: 5/5 (100%) — ESTAVEL**
**Notas:** "reais" + valor numérico é um sinal forte para o classificador financeiro.

---

### Grupo C1 — Agenda que parece conversa
**Mensagem:** `"Preciso lembrar de ir no dentista sexta"`
**Armadilha:** Tom de desabafo/conversa, não de comando. "Preciso lembrar" pode ir pra branch padrão.
**Intenção esperada:** Agenda (criar evento)

| # | Horário | Resposta | Veredicto |
|---|---------|----------|-----------|
| 1 | 14:20:03 | Evento agendado! Ir no Dentista 03/04 às 09:00 | PASS |
| 2 | 14:20:16 | Evento agendado! Ir no Dentista 03/04 às 09:00 | PASS |
| 3 | 14:20:29 | Evento agendado! Ir no Dentista 03/04 às 09:00 | PASS |
| 4 | 14:20:40 | Evento agendado! Ir no Dentista 03/04 às 09:00 | PASS |
| 5 | 14:20:54 | Evento agendado! Ir no Dentista 03/04 às 09:00 | PASS |

**Taxa: 5/5 (100%) — ESTAVEL**
**Notas:** "sexta" (dia da semana) é sinal forte para agenda. Classificador não se confundiu com o tom conversacional.

---

### Grupo X1 — Financeiro que parece agenda
**Mensagem:** `"Almocei hoje 45 reais"`
**Armadilha:** "hoje" = elemento temporal que pode puxar pra agenda
**Intenção esperada:** Financeiro (registrar gasto)

| # | Horário | Resposta | Veredicto |
|---|---------|----------|-----------|
| 1 | 14:21:05 | Gasto registrado! Almoço R$45 Alimentação | PASS |
| 2 | 14:21:19 | Gasto registrado! Almoço R$45 Alimentação | PASS |
| 3 | 14:21:30 | Gasto registrado! Almoço R$45 Alimentação | PASS |
| 4 | 14:21:43 | Gasto registrado! Almoço R$45 Alimentação | PASS |
| 5 | 14:21:55 | Gasto registrado! Almoço R$45 Alimentação | PASS |

**Taxa: 5/5 (100%) — ESTAVEL**
**Notas:** "reais" + número domina sobre "hoje". Classificador financeiro é robusto.

---

### Grupo Z1 — Zona cinzenta total
**Mensagem:** `"Café com a Ana 15 reais segunda"`
**Armadilha:** Contém AMBOS os sinais: "15 reais" (financeiro) + "segunda" (agenda). Genuinamente multi-intenção.
**Intenção esperada:** Ambíguo — gasto E/OU agenda seriam respostas válidas

| # | Horário | Resposta | Branch escolhida | Veredicto |
|---|---------|----------|-----------------|-----------|
| 1 | 14:22:10 | "Essa mensagem não parece ser sobre agenda." | AGENDA (rejeitou) | FAIL |
| 2 | 14:22:21 | "Essa mensagem não parece ser sobre agenda." | AGENDA (rejeitou) | FAIL |
| 3 | 14:22:34 | Evento agendado! Café com Ana 30/03 às 09:00 | AGENDA (aceitou) | PARTIAL |
| 4 | 14:22:45 | Evento agendado! Café com Ana 30/03 às 09:00 | AGENDA (aceitou) | PARTIAL |
| 5 | 14:22:58 | Evento agendado! Café com a Ana 30/03 às 09:00 | AGENDA (aceitou) | PARTIAL |

**Taxa: 3/5 agenda (60%) — INSTAVEL**
**Notas criticas:**
- O classificador SEMPRE roteou para AGENDA, nunca para financeiro
- Mesmo roteando para agenda, 2/5 vezes o Agent rejeitou ("não parece ser sobre agenda")
- O valor "15 reais" foi **completamente ignorado** em todas as 5 tentativas — nenhum gasto registrado
- Multi-intenção confirmada como ponto cego: sistema perde uma das intenções 100% das vezes

---

## Resumo Consolidado

| Grupo | Mensagem | Intenção | Taxa PASS | Consistência | Status |
|-------|----------|----------|-----------|-------------|--------|
| Piloto | "Enviar mensagem Gustavo Garcia as 10h30" | Agenda | 2/5 (40%) | INSTAVEL | BUG |
| A1 | "Ligar pro João às 15h" | Agenda | 5/5 (100%) | ESTAVEL | OK |
| F1 | "Paguei o almoço pro Gustavo 50 reais" | Financeiro | 5/5 (100%) | ESTAVEL | OK |
| C1 | "Preciso lembrar de ir no dentista sexta" | Agenda | 5/5 (100%) | ESTAVEL | OK |
| X1 | "Almocei hoje 45 reais" | Financeiro | 5/5 (100%) | ESTAVEL | OK |
| Z1 | "Café com a Ana 15 reais segunda" | Ambos | 3/5 (60%) | INSTAVEL | BUG |

---

## Diagnostico — Onde o Classificador Falha

### Padrão identificado

O classificador funciona bem quando a mensagem tem **sinais fortes e unidirecionais**:
- Horário explícito ("às 15h") + ação → AGENDA (100%)
- Valor + "reais" → FINANCEIRO (100%)
- Dia da semana + ação → AGENDA (100%)

O classificador **falha** quando:
1. **"Enviar mensagem"** — o verbo "enviar mensagem" compete com a intenção de agenda. A palavra "mensagem" confunde o classificador para a branch padrão/genérica.
2. **Sinais conflitantes** — "15 reais" (financeiro) + "segunda" (agenda) na mesma frase causa indeterminismo. O classificador escolhe agenda mas o Agent não tem certeza.

### Camada da falha

| Caso | Camada | Causa raiz |
|------|--------|------------|
| Piloto | CAMADA 1 — CLASSIFICADOR | "Enviar mensagem" confunde Escolher Branch. Oscila entre `padrao` e `agenda`. |
| Z1 | CAMADA 1 — CLASSIFICADOR + CAMADA 2 — AI AGENT | Classificador acerta branch (agenda), mas o Agent dentro da branch rejeita por insegurança. Multi-intenção = ponto cego. |

### Observacao sobre Z1

O Z1 revela um segundo nível de problema:
- O **Classificador** escolhe agenda (correto — tem "segunda")
- Mas o **AI Agent** dentro da branch agenda recebe "15 reais" e fica confuso
- Em 2/5 vezes, o Agent decide que "não parece ser sobre agenda" — **contradizendo o próprio classificador**
- Em nenhuma das 5 tentativas o gasto de R$15 foi registrado

---

## Bugs Encontrados

| ID | Severidade | Descrição | Frequência |
|----|-----------|-----------|------------|
| BUG-CLASS-01 | ALTA | "Enviar mensagem [pessoa] às [hora]" oscila entre branch padrão e agenda (40% acerto) | 3/5 falhas |
| BUG-CLASS-02 | MEDIA | Mensagens com sinais conflitantes (valor + dia) causam indeterminismo no Agent | 2/5 falhas |
| BUG-MULTI-01 | MEDIA | Multi-intenção (gasto + agenda) sempre perde uma intenção (gasto ignorado 100%) | 5/5 perde gasto |
| BUG-AGENT-01 | MEDIA | AI Agent pode contradizer o Classificador ("não parece ser sobre agenda" após Escolher Branch ter roteado para agenda) | 2/5 ocorrências |

---

## Recomendacoes

| # | Acao | Impacto | Prioridade |
|---|------|---------|------------|
| 1 | Adicionar "enviar mensagem", "mandar msg", "avisar" como sinônimos de agenda no prompt do Classificador | Corrige BUG-CLASS-01 | ALTA |
| 2 | Implementar detector de multi-intenção antes do Classificador | Corrige BUG-MULTI-01 | MEDIA |
| 3 | Alinhar AI Agent com decisão do Classificador (se Branch=agenda, Agent não pode rejeitar como "não é agenda") | Corrige BUG-AGENT-01 | MEDIA |
| 4 | Testar frases adicionais do tipo "verbo ambíguo + pessoa + horário" para mapear mais edge cases | Ampliar cobertura | BAIXA |

---

## Proximos Passos Sugeridos

1. Rodar testes A2-A5 (demais verbos ambíguos) para mapear quais verbos causam instabilidade
2. Investigar o prompt do Escolher Branch no Fix Conflito v2 para entender as regras de classificação
3. Testar F2-F5 para verificar se "dividi", "dei", "deu", "coloquei" são reconhecidos como financeiro
4. Executar diagnóstico completo (5 camadas) no BUG-CLASS-01 com análise da execução N8N

---

*— Lupa, constatando com precisao*
