# Fase 2 — Diagnostico + Proposta de Solucao

**Data:** 2026-03-27
**Auditor:** Lupa (Inspector)

---

## 1. Anatomia do Problema

O sistema tem **2 camadas de decisao** em serie:

```
Mensagem do user
    ↓
[CAMADA 1] Classificador (01-classificador.txt)
    → Decide o branch: criar_evento_agenda, criar_gasto, padrao, etc.
    ↓
[CAMADA 2] Agent do branch (ex: prompt_criar1)
    → Executa a acao OU rejeita ("nao parece ser sobre agenda")
    ↓
Resposta ao user
```

**O bug esta nas DUAS camadas, por motivos diferentes:**

### Camada 1 — Classificador

**Regra R5 (Agenda):**
> "Se mensagem contiver QUALQUER ACAO/SUBSTANTIVO + HORARIO/DATA → criar_evento_agenda"
> "QUALQUER texto + horario = evento. Nao julgue se 'parece' agenda."

Essa regra e boa em teoria, mas o LLM (GPT-4.1-mini) **nao a segue 100%** porque:
- A palavra "mensagem" / "enviar mensagem" ativa padroes internos do LLM de "isso e sobre comunicacao, nao agenda"
- O LLM tem bias semantico — ele "julga" mesmo quando o prompt diz "nao julgue"
- Com sinais conflitantes ("15 reais" + "segunda"), o LLM oscila entre R4 (financeiro) e R5 (agenda)

### Camada 2 — Agent do Branch

**prompt_criar1 tem esta regra:**
> "Fora de agenda (financas, compras, precos) → acao='padrao', tool=[], mensagem='Essa mensagem nao parece ser sobre agenda.'"

Entao MESMO quando o Classificador acerta e manda pra `criar_evento_agenda`, o Agent dentro da branch pode **contradizer** e rejeitar, porque:
- Ele ve "15 reais" ou "mensagem" e ativa a regra de "fora de agenda"
- Ele tem autonomia para dizer "nao parece ser sobre agenda" — o que cria um **conflito Classificador vs Agent**

---

## 2. Diagnostico por Camada (5 Camadas)

| Camada | Status | Problema |
|--------|--------|----------|
| 1 — CLASSIFICADOR | **FALHA INTERMITENTE** | R5 diz "qualquer texto + horario = agenda" mas o LLM tem bias semantico que ignora essa regra em ~60% dos casos com verbos como "enviar mensagem" |
| 2 — AI AGENT | **FALHA INTERMITENTE** | prompt_criar1 tem regra de rejeicao ("nao parece ser sobre agenda") que contradiz a decisao do classificador |
| 3 — TOOL HTTP | OK | Quando o Agent aceita, o webhook secundario funciona |
| 4 — SUPABASE | OK | Dados gravados corretamente quando a tool executa |
| 5 — RESPOSTA | OK | Formato da resposta correto quando criado |

**Causa raiz:** CLASSIFICACAO_ERRADA + RESPOSTA_ERRADA (conflito entre camadas)

---

## 3. Proposta de Solucao

### Filosofia

**Menos deterministico, mais resiliente.** Em vez de bloquear quando ha duvida, o sistema deve:

1. **Seguir com o mais provavel** — sempre executar a acao mais provavel
2. **Educar o usuario** — quando a confianca for baixa, adicionar uma dica gentil
3. **Nunca rejeitar sem motivo forte** — "nao parece ser sobre agenda" so quando REALMENTE nao e

### 3.1 — Mudanca no Classificador (01-classificador.txt)

#### Problema atual da R5:
```
## R5 — AGENDA (criar)
Se mensagem contiver QUALQUER ACAO/SUBSTANTIVO + HORARIO/DATA → criar_evento_agenda.
QUALQUER texto + horario = evento. Nao julgue se "parece" agenda.
```

#### R5 proposta (v2):
```
## R5 — AGENDA (criar)
Se mensagem contiver QUALQUER texto + HORARIO/DATA → criar_evento_agenda.
QUALQUER texto + horario = evento. NAO JULGUE o verbo ou o contexto.

VERBOS QUE SAO AGENDA (exemplos — a lista NAO e exaustiva):
"enviar mensagem", "mandar msg", "avisar", "ligar", "falar com",
"passar no", "buscar", "pegar", "levar", "ir no", "ver o/a",
"lembrar de", "nao esquecer"
→ Se tem horario/data junto, e SEMPRE criar_evento_agenda.

REGRA DE OURO: Se tem horario ou data na mensagem e NAO tem valor
em reais/R$, e criar_evento_agenda. Ponto final.
```

#### Nova R4.5 — Desempate financeiro vs agenda:
```
## R4.5 — DESEMPATE (quando tem VALOR + DATA/HORARIO na mesma mensagem)
Se a mensagem contiver VALOR (reais/R$/numero) E DATA/HORARIO:
  → Se o verbo e financeiro ("gastei", "paguei", "comprei", "custou") → criar_gasto
  → Se o verbo e de acao ("cafe com", "almoco com", "reuniao") → criar_evento_agenda
  → Se ambiguo → criar_gasto (dinheiro e mais urgente de registrar)
Exemplo: "Cafe com a Ana 15 reais segunda"
  → Tem "15 reais" (financeiro) + "segunda" (agenda)
  → Verbo e "cafe com" (acao social) → criar_evento_agenda
  → MAS o classificador NAO resolve multi-intencao. Escolha o DOMINANTE.
```

---

### 3.2 — Mudanca no prompt_criar1 (Agent de Agenda)

#### Problema atual:
```
Fora de agenda (financas, compras, precos) → acao="padrao", tool=[],
mensagem="Essa mensagem nao parece ser sobre agenda."
```

Esta regra e a **causa direta** do BUG-AGENT-01. O Agent tem poder de veto sobre o Classificador.

#### Proposta (v2) — Confianca no Classificador + Dica educativa:

**Remover a rejeicao dura.** Substituir por:

```
REGRA DE CONFIANCA NO CLASSIFICADOR:
Se voce esta recebendo esta mensagem, o Classificador JA decidiu que e agenda.
NAO questione essa decisao. SEMPRE tente criar o evento.

Se a mensagem parecer ambigua (ex: contem valor em reais, ou verbo incomum):
1. CRIE o evento mesmo assim (siga com o mais provavel)
2. Adicione uma dica na mensagem:
   "✅ Evento agendado!\n📅 [Nome] ⏰ [Data/Hora]\n\n💡 Dica: para eu te
   entender melhor, tente algo como 'reuniao com Gustavo amanha 10h30'."

NUNCA responda com "Essa mensagem nao parece ser sobre agenda."
Se genuinamente nao conseguir extrair nome NEM data → peca esclarecimento:
"Nao consegui identificar a data. Quando seria?"
```

---

### 3.3 — Mensagem de Dica (UX)

Quando a confianca for baixa (mensagem ambigua), o sistema adiciona uma **dica educativa** sem bloquear:

```
✅ Evento agendado!
📅 Enviar Mensagem Gustavo Garcia ⏰ hoje às 10:30

💡 Dica: para aproveitar o máximo do seu Total Assistente,
tente me dar instruções mais diretas, tipo:
"reunião com Gustavo 10h30" ou "ligar pro João amanhã 15h"
```

**Regras da dica:**
- Aparece NO MAXIMO 1 vez a cada 5 mensagens do user (nao ser chato)
- So aparece quando o Agent detecta ambiguidade (verbo incomum, sinais mistos)
- NAO aparece em mensagens claras ("reuniao amanha 14h" → sem dica)
- Tom: gentil, util, nunca critico

---

### 3.4 — Logica de Confianca (opcional, mais sofisticado)

Adicionar um campo `confianca` no output do Classificador:

```json
{ "branch": "criar_evento_agenda", "confianca": "alta" }
{ "branch": "criar_evento_agenda", "confianca": "media" }
```

**Quando usar:**
- `alta` → sinais claros, sem ambiguidade → Agent cria SEM dica
- `media` → sinais mistos ou verbo incomum → Agent cria COM dica educativa
- O Agent NUNCA rejeita em nenhum caso

**Como o Classificador decide a confianca:**
```
confianca = "alta" se:
  - Horario explicito + verbo claro de agenda (reuniao, consulta, treino)
  - Valor explicito + verbo financeiro (gastei, paguei, comprei)

confianca = "media" se:
  - Horario presente mas verbo ambiguo (enviar, mandar, falar)
  - Sinais de multiplos dominios na mesma mensagem
  - Mensagem muito curta sem contexto claro
```

---

## 4. Resumo das Mudancas

| # | Arquivo | Mudanca | Resolve |
|---|---------|---------|---------|
| 1 | `01-classificador.txt` | Expandir R5 com lista de verbos-agenda + Regra de Ouro | BUG-CLASS-01 |
| 2 | `01-classificador.txt` | Adicionar R4.5 (desempate valor + data) | BUG-CLASS-02 |
| 3 | `prompt_criar1` | Remover rejeicao "nao parece ser sobre agenda" | BUG-AGENT-01 |
| 4 | `prompt_criar1` | Adicionar dica educativa quando confianca e media | UX |
| 5 | `01-classificador.txt` (opcional) | Campo `confianca` no output JSON | UX avancado |

---

## 5. O que NAO mudar

- **R1 a R4** do classificador estao funcionando bem (100% nos testes)
- **R7 a R13** estao estaveis
- **prompt_criar1** — toda a logica de datas, duracao, parsing esta correta
- **Formato de output** — manter JSON puro

---

## 6. Plano de Implementacao

### Fase 1 — Quick Fix (10 min)
1. Atualizar R5 no `01-classificador.txt` com lista de verbos
2. Remover "Essa mensagem nao parece ser sobre agenda" do `prompt_criar1`
3. Adicionar regra de confianca no classificador (Agent nunca rejeita)

### Fase 2 — Dica Educativa (15 min)
4. Adicionar template de dica no `prompt_criar1`
5. Definir regra de frequencia (max 1 a cada 5 mensagens)

### Fase 3 — Validacao (20 min)
6. Re-rodar os 30 testes originais (piloto + bateria)
7. Comparar taxa antes/depois
8. Meta: 100% nos testes que falharam (A0 piloto, Z1)

### Fase 4 — Irradiacao (30 min)
9. Rodar os 70 testes restantes (A2-A5, F2-F5, C2-C5, X2-X3, Z2-Z3)
10. Documentar novos edge cases se encontrados

---

## 7. Prompts Propostos (Prontos para Copiar)

### 01-classificador.txt — Trecho R5 atualizado

```
## R5 — AGENDA (criar)
Se mensagem contiver QUALQUER texto + HORARIO/DATA → criar_evento_agenda.
NAO JULGUE o verbo, o contexto ou se "parece" agenda. Se tem horario, e agenda.

EXEMPLOS DE VERBOS QUE SAO AGENDA (lista nao exaustiva):
"enviar mensagem", "mandar msg", "avisar", "ligar", "falar com",
"passar no", "buscar", "pegar", "levar", "ir no", "ver",
"lembrar de", "nao esquecer", "preciso", "tenho que"
→ Se acompanhados de horario/data = SEMPRE criar_evento_agenda.

REGRA DE OURO: Se tem horario/data E NAO tem valor monetario (reais/R$/numero seguido de "reais"), e criar_evento_agenda. Sem excecao.

Exemplos:
"enviar mensagem Gustavo Garcia as 10h30" → criar_evento_agenda
"ligar pro Joao as 15h" → criar_evento_agenda
"falar com Maria amanha as 9h" → criar_evento_agenda
"passar no mercado sexta as 18h" → criar_evento_agenda
"buscar as criancas amanha 11h30" → criar_evento_agenda
"massagear os pes do amaral 19h" → criar_evento_agenda
```

### 01-classificador.txt — Nova R4.5

```
## R4.5 — DESEMPATE (VALOR + DATA na mesma mensagem)
Se a mensagem contiver VALOR (reais/R$/numero) E HORARIO/DATA simultaneamente:
  → Verbo financeiro explicito ("gastei", "paguei", "comprei", "custou", "dei") → criar_gasto
  → Qualquer outro caso → criar_evento_agenda (agenda e mais provavel quando tem horario)
Exemplos:
"Cafe com a Ana 15 reais segunda" → criar_evento_agenda (acao + dia > valor)
"gastei 50 reais no almoco hoje" → criar_gasto (verbo financeiro explicito)
```

### prompt_criar1 — Trecho de rejeicao REMOVIDO e substituido

```
REMOVER:
  Fora de agenda (financas, compras, precos) → acao="padrao", tool=[], mensagem="Essa mensagem nao parece ser sobre agenda."

SUBSTITUIR POR:
  REGRA DE CONFIANCA: O Classificador ja decidiu que esta mensagem e agenda.
  SEMPRE tente criar o evento. NUNCA responda "nao parece ser sobre agenda".

  Se a mensagem parecer ambigua mas tiver nome inferivel + data resolvivel:
  1. CRIE o evento normalmente
  2. Adicione dica na mensagem (1 vez a cada 5 msgs, nao toda vez):
     "\n\n💡 Para aproveitar o maximo do seu Total Assistente, tente algo como: 'reuniao com Gustavo amanha 10h30'"

  Se GENUINAMENTE nao conseguir extrair nome NEM data:
  → acao="padrao", mensagem="Nao consegui identificar o evento. Pode me dizer o que e quando?"
```

---

*— Lupa, constatando com precisao*
