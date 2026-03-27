# Fase 3 — Irradiacao

**Data:** 2026-03-27
**Objetivo:** Testar se o padrao de instabilidade do classificador se repete em outras mensagens linguisticamente ambiguas.

## Padrao identificado

**Bug original:** A palavra "mensagem" no input compete com sinais de agenda ("as 10h30"), causando roteamento inconsistente.

**Padrao generalizado:** Mensagens onde a intencao real do usuario difere das palavras-chave que o classificador espera.

## Testes de irradiacao — 5 grupos, 5 repeticoes cada

### Grupo A1 — Agenda com verbo ambiguo
**Mensagem:** `Ligar pro Joao as 15h`
**Armadilha:** "ligar" nao soa como "agendar evento"

| # | Resposta | Veredicto |
|---|----------|-----------|
| 1 | Evento agendado! Ligar para Joao hoje as 15:00 | PASS |
| 2 | Evento agendado! Ligar para Joao hoje as 15:00 | PASS |
| 3 | Evento agendado! Ligar para Joao hoje as 15:00 | PASS |
| 4 | Evento agendado! Ligar para Joao hoje as 15:00 | PASS |
| 5 | Evento agendado! Ligar para Joao hoje as 15:00 | PASS |

**Taxa: 5/5 (100%) — ESTAVEL** | Mesmo bug? NAO

---

### Grupo F1 — Financeiro com contexto social
**Mensagem:** `Paguei o almoco pro Gustavo 50 reais`
**Armadilha:** "paguei pro Gustavo" parece interpessoal

| # | Resposta | Veredicto |
|---|----------|-----------|
| 1 | Gasto registrado! Almoco pro Gustavo R$50 Alimentacao | PASS |
| 2 | Gasto registrado! Almoco pro Gustavo R$50 Alimentacao | PASS |
| 3 | Gasto registrado! Almoco pro Gustavo R$50 Alimentacao | PASS |
| 4 | Gasto registrado! Almoco pro Gustavo R$50 Alimentacao | PASS |
| 5 | Gasto registrado! Almoco pro Gustavo R$50 Alimentacao | PASS |

**Taxa: 5/5 (100%) — ESTAVEL** | Mesmo bug? NAO

---

### Grupo C1 — Agenda que parece conversa
**Mensagem:** `Preciso lembrar de ir no dentista sexta`
**Armadilha:** Tom de desabafo, nao de comando

| # | Resposta | Veredicto |
|---|----------|-----------|
| 1 | Evento agendado! Ir no Dentista 03/04 as 09:00 | PASS |
| 2 | Evento agendado! Ir no Dentista 03/04 as 09:00 | PASS |
| 3 | Evento agendado! Ir no Dentista 03/04 as 09:00 | PASS |
| 4 | Evento agendado! Ir no Dentista 03/04 as 09:00 | PASS |
| 5 | Evento agendado! Ir no Dentista 03/04 as 09:00 | PASS |

**Taxa: 5/5 (100%) — ESTAVEL** | Mesmo bug? NAO

---

### Grupo X1 — Financeiro que parece agenda
**Mensagem:** `Almocei hoje 45 reais`
**Armadilha:** "hoje" = elemento temporal que pode confundir para agenda

| # | Resposta | Veredicto |
|---|----------|-----------|
| 1 | Gasto registrado! Almoco R$45 Alimentacao | PASS |
| 2 | Gasto registrado! Almoco R$45 Alimentacao | PASS |
| 3 | Gasto registrado! Almoco R$45 Alimentacao | PASS |
| 4 | Gasto registrado! Almoco R$45 Alimentacao | PASS |
| 5 | Gasto registrado! Almoco R$45 Alimentacao | PASS |

**Taxa: 5/5 (100%) — ESTAVEL** | Mesmo bug? NAO

---

### Grupo Z1 — Zona cinzenta (multi-intencao)
**Mensagem:** `Cafe com a Ana 15 reais segunda`
**Armadilha:** Contem AMBOS sinais: "15 reais" (financeiro) + "segunda" (agenda)

| # | Resposta | Branch | Veredicto |
|---|----------|--------|-----------|
| 1 | "Essa mensagem nao parece ser sobre agenda." | Agenda (rejeitou) | FAIL |
| 2 | "Essa mensagem nao parece ser sobre agenda." | Agenda (rejeitou) | FAIL |
| 3 | Evento agendado! Cafe com Ana 30/03 as 09:00 | Agenda (aceitou) | PARTIAL |
| 4 | Evento agendado! Cafe com Ana 30/03 as 09:00 | Agenda (aceitou) | PARTIAL |
| 5 | Evento agendado! Cafe com a Ana 30/03 as 09:00 | Agenda (aceitou) | PARTIAL |

**Taxa agenda: 3/5 (60%) — INSTAVEL** | Mesmo bug? SIM (variante)
**Gasto registrado:** 0/5 — multi-intencao perdida 100%

---

## Tabela consolidada de irradiacao

| Grupo | Mensagem | Taxa | Mesmo bug? | Variante? | Impacto |
|-------|----------|------|-----------|-----------|---------|
| A1 | "Ligar pro Joao as 15h" | 5/5 (100%) | NAO | — | — |
| F1 | "Paguei o almoco pro Gustavo 50 reais" | 5/5 (100%) | NAO | — | — |
| C1 | "Preciso lembrar de ir no dentista sexta" | 5/5 (100%) | NAO | — | — |
| X1 | "Almocei hoje 45 reais" | 5/5 (100%) | NAO | — | — |
| Z1 | "Cafe com a Ana 15 reais segunda" | 3/5 (60%) | SIM | Sinais conflitantes | ALTO |

## Classificacao do impacto

**LOCALIZADO** — O bug se manifesta em 2 cenarios identificados:
1. Mensagens com a palavra "mensagem/msg/enviar mensagem" + horario (bug original)
2. Mensagens com sinais conflitantes de multiplas branches (variante Z1)

O classificador e estavel quando os sinais sao unidirecionais (verbo + horario OU valor + "reais").

## Testes sugeridos para proxima irradiacao

| # | Mensagem | Tipo de armadilha |
|---|----------|-------------------|
| A2 | "Falar com Maria amanha as 9h" | Verbo social + horario |
| A3 | "Passar no mercado sexta as 18h" | Verbo de acao + dia |
| A4 | "Buscar as criancas amanha 11h30" | Verbo de tarefa + horario |
| Z2 | "Almoco com cliente amanha 12h" | Agenda com contexto que poderia ser gasto |
| Z3 | "Aniversario do Joao sabado, comprar presente 100 reais" | Multi-intencao explicita |
