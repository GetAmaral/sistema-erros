# Diff Completo — O que mudou (v2 atual → v4.1)

**Arquivos afetados:** 2
1. `01-classificador.txt` (Escolher Branch) — **reescrito**
2. `prompt_criar1` (Set node de criação de evento) — **2 linhas alteradas**

---

## ARQUIVO 1: 01-classificador.txt

### Mudanças estruturais

| Aspecto | v2 (atual no N8N) | v4.1 (nova) |
|---------|-------------------|-------------|
| Quantidade de regras | 13 (R1-R13) | 14 (R1-R14) |
| Ordem R4/R5 | R4=financeiro, R5=agenda | R4=relatório, R5=busca (invertido) |
| Relatórios | R11 (baixa prioridade) | R4 (alta prioridade) |
| Busca | R7 (baixa prioridade) | R5 (alta prioridade) |
| Edição | R8 (baixa prioridade) | R6 (alta prioridade) |
| Exclusão | R9 (baixa prioridade) | R7 (alta prioridade) |
| Criação financeiro | R4 (alta prioridade) | R9 e R12 (dividido em 2) |
| Criação agenda | R5 (alta prioridade) | R11 (baixa prioridade) |
| Recorrente | R6 | R10 |
| Fluxograma | Não existia | Adicionado (12 passos) |
| Exemplos | ~12 | ~50+ |
| Branch criar_lembrete | Não existia (dead code) | Confirmado removido |

### DIFF: R4 — FINANCEIRO (MUDOU)

```diff
- ## R4 — FINANCEIRO (criar)
- Se mensagem contiver NOME + VALOR/DINHEIRO (R$, reais, número com vírgula/ponto) → criar_gasto.
- Exemplos: "uber 27,90" | "mercado 150" | "farmácia 45,50" | "paguei aluguel 1200"

+ ## R4 — RELATÓRIOS
+ "relatório semanal" / "resumo da semana" / "relatório dessa semana" → relatorio_semanal
+ "relatório mensal" / "relatório do mês" / "resumo do mês" → relatorio_mensal
```

**O que aconteceu:** A regra de financeiro foi DIVIDIDA em duas (R9 e R12) e REBAIXADA na prioridade. Relatórios subiram para R4.

**Por que:** A regra antiga "NOME + VALOR → gasto" capturava TUDO com valor, mesmo mensagens com data que deveriam ser agenda. Ex: "pagar 100 amanhã pro Luan" ia pro gasto, mas o user queria agendar.

---

### DIFF: R5 — AGENDA (MUDOU)

```diff
- ## R5 — AGENDA (criar)
- Se mensagem contiver QUALQUER AÇÃO/SUBSTANTIVO + HORÁRIO/DATA → criar_evento_agenda.
- QUALQUER texto + horário = evento. Não julgue se "parece" agenda.
- Exemplos: "massagear os pés do amaral 19h" → criar_evento_agenda | "senai presencial 16 de abril"
-   → criar_evento_agenda | "cinema 15h30" → criar_evento_agenda | "futebol 20h"
-   → criar_evento_agenda | "consulta dentista amanhã 8:30" → criar_evento_agenda |
-   "me lembra de pagar água amanhã" → criar_evento_agenda

+ ## R5 — BUSCA EXPLÍCITA
+ Se o user quer VER/CONSULTAR dados existentes (NÃO criar, NÃO editar, NÃO excluir).
+
+ Verbos de busca: "quanto gastei", "meus gastos", "me mostra", "lista", "quais",
+ "o que tem", "o que tenho", "agenda de", "quero ver", "como estão"
+
+ ATENÇÃO — "buscar" e "procurar" são AMBÍGUOS:
+   "buscar meus gastos" → CONSULTA → buscar ✓
+   "buscar crianças 11h30" → AÇÃO FÍSICA → NÃO é busca ✗
+ REGRA: "buscar/procurar" só é busca se seguido de DADOS:
+   gastos, registros, eventos, agenda, compromissos, informações, histórico, extrato
+ Se seguido de PESSOA/OBJETO/LUGAR + horário → NÃO é busca. Pular para R11.
+
+ - Busca financeira → buscar
+ - Busca agenda → buscar_evento_agenda
```

**O que aconteceu:** A criação de agenda foi REBAIXADA para R11. Busca explícita subiu para R5.

**Por que:** Na v2, busca (R7) vinha DEPOIS de agenda (R5). Mas a nova regra "TEM DATA → agenda" (R11) engolia buscas como "quanto gastei hoje?" e "o que tenho amanhã?". Busca precisa ser checada ANTES.

---

### DIFF: R6 — RECORRENTE → EDIÇÃO (MUDOU)

```diff
- ## R6 — AGENDA (recorrente)
- Se mencionar repetição ("todo dia", "toda segunda", "toda semana", "todo mês",
-   "de X em X") → criar_evento_recorrente.

+ ## R6 — EDIÇÃO EXPLÍCITA
+ Se o user quer MUDAR algo que JÁ EXISTE:
+ Verbos de edição: "muda", "troca", "altera", "corrige", "atualiza", "modifica",
+   "reagenda", "remarca"
```

**O que aconteceu:** Recorrente foi para R10. Edição subiu de R8 para R6.

---

### DIFF: R7 — BUSCA → EXCLUSÃO (MUDOU + EXPANDIDO)

```diff
- ## R7 — BUSCA (só se EXPLÍCITO)
- Somente se o user usar VERBOS DE BUSCA: "buscar", "me mostra", "lista", "quais",
-   "o que tem", "agenda de hoje", "quanto gastei", "meus gastos".
- - Busca agenda → buscar_evento_agenda
- - Busca financeiro → buscar

+ ## R7 — EXCLUSÃO / CANCELAMENTO
+ Se o user quer APAGAR, CANCELAR ou DESISTIR de algo:
+
+ Verbos de exclusão: "cancela", "apaga", "deleta", "remove", "exclui", "tira"
+
+ Negações que indicam cancelamento:
+ "não vou mais", "desisti", "não quero mais", "não preciso mais",
+ "deixa pra lá", "esquece", "esquece o/a", "não vai rolar",
+ "deixa quieto", "mudou de ideia", "não vai dar", "não rola mais"
```

**O que aconteceu:** Busca foi para R5. Exclusão subiu de R9 para R7 e ganhou 8 negações novas.

**Negações adicionadas (NÃO existiam na v2):**
- "não preciso mais"
- "deixa pra lá"
- "esquece" / "esquece o/a"
- "não vai rolar"
- "deixa quieto"
- "mudou de ideia"
- "não vai dar"
- "não rola mais"

---

### DIFF: R8 — EDIÇÃO → LIMITES (MUDOU posição)

```diff
- ## R8 — EDIÇÃO
- Se user falar em mudar/trocar/corrigir/editar algo JÁ EXISTENTE:
- - Agenda → editar_evento_agenda
- - Financeiro → editar

+ ## R8 — LIMITES
+ "criar limite" → criar_limite | "mudar/editar limite" → editar_limite |
+ "remover/excluir limite" → excluir_limite
```

**O que aconteceu:** Edição subiu para R6. Limites tomou o lugar de R8.

---

### DIFF: R9 — EXCLUSÃO → GASTO PASSADO (NOVO)

```diff
- ## R9 — EXCLUSÃO
- Se user falar em apagar/deletar/remover/excluir/tirar:
- - Agenda → excluir_evento_agenda
- - Financeiro → excluir

+ ## R9 — GASTO NO PASSADO (registro de algo que JÁ aconteceu)
+ Se mensagem tiver VERBO NO PASSADO + VALOR → criar_gasto.
+ MESMO que tenha data na mensagem — se o verbo é passado, o user está registrando.
+
+ Verbos no passado: "gastei", "paguei", "comprei", "almocei", "jantei", "dei",
+   "custou", "foi", "ficou", "saiu", "deu" (seguido de valor)
```

**O que aconteceu:** Exclusão subiu para R7. R9 agora é gasto com verbo passado — regra NOVA que não existia na v2.

**Por que:** Na v2, "gastei 80 no jantar hoje" podia ser capturado pela R4 genérica (valor → gasto) OU pela R5 (hoje → agenda). Agora, verbo passado + valor SEMPRE vai pra gasto, independente de ter data.

---

### DIFF: R10 — LIMITES → RECORRENTE (MUDOU + EXPANDIDO)

```diff
- ## R10 — LIMITES
- "criar limite" → criar_limite | "mudar/editar limite" → editar_limite |
-   "remover/excluir limite" → excluir_limite

+ ## R10 — AGENDA RECORRENTE
+ Se mencionar repetição EXPLÍCITA:
+ "todo dia", "toda segunda", "toda semana", "todo mês", "de X em X", "sempre às"
+ → criar_evento_recorrente
+
+ Recorrência IMPLÍCITA — se a mensagem listar 2 ou mais dias da semana + horário:
+ "academia segunda quarta e sexta 6h" → criar_evento_recorrente
+ "inglês terça e quinta 19h" → criar_evento_recorrente
+ "pilates segunda e quarta 7h" → criar_evento_recorrente
+
+ REGRA: 2+ dias da semana na mesma mensagem + horário = recorrente. Sem exceção.
```

**O que aconteceu:** Limites foi para R8. Recorrente ganhou detecção IMPLÍCITA (nova, não existia na v2).

**Adicionado:** Detecção de recorrência sem "todo/toda" — quando 2+ dias da semana aparecem na mesma mensagem.

---

### DIFF: R11 — RELATÓRIOS → CRIAÇÃO AGENDA (NOVO)

```diff
- ## R11 — RELATÓRIOS
- "relatório semanal" / "resumo da semana" → relatorio_semanal
- "relatório mensal" / "relatório do mês" → relatorio_mensal

+ ## R11 — CRIAÇÃO: TEM DATA/HORA → AGENDA
+ Se a mensagem contiver HORÁRIO ou DATA e NENHUMA regra acima pegou:
+ → criar_evento_agenda
+
+ (... 18 exemplos incluindo lembretes, verbos incomuns, valores ...)
```

**O que aconteceu:** Relatórios subiu para R4. R11 é a nova regra de criação de agenda — catch-all para qualquer mensagem com data/hora que não foi capturada antes.

---

### DIFF: R12 — NÃO EXISTIA → GASTO SEM DATA (NOVO)

```diff
+ ## R12 — CRIAÇÃO: SEM DATA + TEM VALOR → GASTO
+ Se a mensagem NÃO tem data/hora MAS tem VALOR → criar_gasto.
+ NÃO importa o tempo verbal (presente ou futuro):
+ "vou pagar 100 na lavação do carro" → criar_gasto
+ "pago 100 pro Luan" → criar_gasto
+ "uber 27,90" → criar_gasto
```

**Regra NOVA.** Na v2, não existia distinção entre gasto com e sem data. Agora:
- Com data → agenda (R11)
- Sem data → gasto (R12)

---

### DIFF: FLUXOGRAMA (NOVO)

```diff
+ # FLUXOGRAMA DECISIVO (consulte se estiver em dúvida)
+ 1. É confirmação/retry? → R1/R2
+ 2. É receita? → R3 (criar_gasto)
+ 3. É relatório? → R4
+ 4. Tem verbo de BUSCA (e não é ação física)? → R5
+ 5. Tem verbo de EDIÇÃO? → R6
+ 6. Tem verbo de EXCLUSÃO ou NEGAÇÃO de cancelamento? → R7
+ 7. É sobre limites? → R8
+ 8. Tem verbo PASSADO + valor? → R9 (criar_gasto)
+ 9. Tem repetição explícita OU 2+ dias da semana + horário? → R10
+ 10. Tem DATA/HORA? → R11 (criar_evento_agenda)
+ 11. Tem VALOR sem data? → R12 (criar_gasto)
+ 12. Nada acima? → R13/R14
```

**Não existia na v2.** Adicionado para o LLM ter uma referência rápida quando estiver em dúvida.

---

## ARQUIVO 2: prompt_criar1 (Set node de criação de evento)

### Mudança: APENAS as 2 primeiras linhas

```diff
  ==Módulo AGENDA — CRIAR EVENTOS. Tudo é EVENTO: avisos, alertas, lembretes,
  compromissos. Diferença = TIPO (compromisso/lembrete) e DURAÇÃO.
- Fora de agenda (finanças, compras, preços) → acao="padrao", tool=[],
- mensagem="Essa mensagem não parece ser sobre agenda."

+ REGRA DE CONFIANÇA: Se você recebeu esta mensagem, o Classificador JÁ decidiu
+ que é agenda. NÃO questione. Seu trabalho é EXTRAIR nome + data/hora e CRIAR o
+ evento. O verbo do user vira o NOME do evento (ex: "enviar mensagem Gustavo"
+ → nome="Enviar Mensagem Gustavo").
+
+ PROIBIDO: A frase "Essa mensagem não parece ser sobre agenda." está BANIDA.
+ Nunca use.
+
+ VERBOS INCOMUNS: O verbo do user pode ser qualquer coisa ("enviar mensagem",
+ "pagar", "comprar", "mandar", "ligar", "falar"). Isso NÃO significa que está
+ fora de agenda. O verbo + complemento vira o NOME do evento.
+
+ VALOR NA MENSAGEM: Se a mensagem contiver valor (R$, reais), o valor entra no
+ NOME do evento entre parênteses. O valor NÃO é registrado como gasto.
+
+ Se GENUINAMENTE não conseguir extrair nome NEM data/hora:
+ → acao="padrao", tool=[], mensagem="Não consegui identificar o evento. Pode me
+ dizer o que e quando?"
+
+ DICA EDUCATIVA (máx 1 a cada 5 mensagens do user):
+ CASO 1 — Verbo de comunicação ("enviar", "mandar", "avisar") + horário:
+   Dica: "\n\n💡 Para aproveitar o máximo do Total Assistente, tente: 'reunião
+   com Gustavo amanhã 10h30'"
+ CASO 2 — Valor monetário presente:
+   Dica: "\n\n💡 Notei que você mencionou R$[valor]. Depois que [verbo passado],
+   me manda: '[nome] [valor] reais' pra eu registrar o gasto."
+ CASO 3 — Nome muito vago:
+   Dica: "\n\n💡 Para eu organizar melhor sua agenda, tente incluir o assunto:
+   'reunião com João 14h'"
+ NÃO MOSTRAR quando mensagem é clara e direta. Nunca 2 dicas seguidas.
```

**Todo o resto do prompt_criar1 permanece INALTERADO:**
- REGRA ZERO (nunca pedir confirmação)
- FORMATO JSON
- DECISÃO criar_evento vs padrao
- compromisso_tipo (compromisso/lembrete)
- DURAÇÃO
- CONTEXTO DE TEMPO
- CALENDÁRIO DINÂMICO
- RESOLUÇÃO DE DATAS
- PARSING DE EVENTOS
- NOME DO EVENTO
- LEMBRETES
- EMOJIS
- REGRAS FINAIS

---

## AÇÃO NO N8N: remover node

```diff
- Node "prompt_lembrete" (Set node que injeta 10-lembrete.txt)
- Output "criar_lembrete_agenda" no Switch (se existir)
```

O prompt_criar1 já trata lembretes via `compromisso_tipo: "lembrete"`. O node prompt_lembrete é dead code.

---

## Resumo visual

```
CLASSIFICADOR (01-classificador.txt):
  ├── R1-R3: SEM MUDANÇA (confirmação, retry, receita)
  ├── R4-R10: REORDENADO (operações explícitas subiram, criação desceu)
  ├── R7: EXPANDIDO (+8 negações)
  ├── R10: EXPANDIDO (+recorrência implícita)
  ├── R5: REESCRITO (desambiguação de "buscar")
  ├── R9: NOVO (gasto passado separado)
  ├── R11: NOVO (catch-all data → agenda)
  ├── R12: NOVO (catch-all valor sem data → gasto)
  └── FLUXOGRAMA: NOVO

PROMPT_CRIAR1:
  ├── Linha 2: REMOVIDA ("não parece ser sobre agenda")
  ├── +17 linhas: Regra de confiança + verbos incomuns + valor no nome + dica educativa
  └── Resto: SEM MUDANÇA

N8N:
  └── Remover node prompt_lembrete (dead code)
```
