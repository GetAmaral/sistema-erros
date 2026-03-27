# Analise de Furos — Classificador v4 vs Todas as Features

**Data:** 2026-03-27
**Metodo:** Testar cada branch contra a v4, cenario por cenario

---

## Branch 1: criar_evento_agenda (R11) — prompt_criar1 / 03-criar-evento

### Funciona bem:
- "reunião com Luan 16h" → R11 ✓
- "enviar msg Gustavo 10h30" → R11 ✓
- "pagar 100 amanhã pro Luan" → R11 ✓

### FURO 1 — "me lembra" vai pra criar_evento_agenda, mas existe branch separado (lembrete)

O N8N tem o prompt `10-lembrete.txt` com branch `criar_lembrete` (acao="criar_lembrete"). Mas no classificador v4, NÃO existe a branch `criar_lembrete` na lista de branches. No prompt original (v2) também não existe.

**Pergunta:** O branch `criar_lembrete` existe no Switch do N8N ou é tratado dentro de `criar_evento_agenda`?

Se existir como branch separado:
- "me lembra de tomar remédio 9h" → R11 manda pra `criar_evento_agenda` → ERRADO, deveria ir pra `criar_lembrete`
- "me avisa daqui 30min" → R11 manda pra `criar_evento_agenda` → ERRADO

Se NÃO existir (lembrete é tratado dentro de criar_evento_agenda):
- O prompt_criar1 e o 03-criar-evento já lidam com compromisso_tipo="lembrete" internamente → OK
- Mas o prompt `10-lembrete.txt` nunca é usado? Verificar no N8N.

**STATUS: PRECISA VERIFICAR no N8N**

---

### FURO 2 — Conflito entre 03-criar-evento.txt e prompt_criar1

Existem DOIS prompts para criar evento:
- `03-criar-evento.txt` — prompt mais simples
- `prompt_criar1-NOVO.md` — prompt completo (v2, o que está em produção?)

Qual dos dois é injetado quando o branch é `criar_evento_agenda`?

**STATUS: PRECISA VERIFICAR no N8N — qual prompt é o ativo?**

---

## Branch 2: criar_gasto (R9/R12) — 04-registrar-gasto

### Funciona bem:
- "gastei 50 no almoço" → R9 (passado + valor) ✓
- "uber 27,90" → R12 (sem data + valor) ✓
- "vou pagar 100 na lavação" → R12 (sem data + valor) ✓
- "mercado 150" → R12 ✓

### FURO 3 — "paguei 1200 do aluguel dia 5" — R9 pega, mas o prompt de gasto aceita datas

O prompt 04-registrar-gasto tem:
> DATA: "hoje"/nada → agora | "ontem" → -1 dia | "amanhã" → +1 dia | data explícita → converter

Então o prompt de gasto ACEITA datas. Se o classificador mandar "paguei 1200 do aluguel dia 5" pro criar_gasto (R9), o prompt vai registrar com data correta. **Funciona.**

Mas e "paguei 1200 do aluguel amanhã"? O verbo é passado ("paguei") mas a data é futura ("amanhã"). Isso é estranho linguisticamente, mas R9 pegaria por causa do verbo passado.

Na prática: ninguém diz "paguei amanhã". Então R9 não pegaria "amanhã" porque o verbo seria "vou pagar" (futuro), não "paguei". Se fosse "vou pagar 1200 do aluguel amanhã" → R11 pega (tem data + não é passado) → vai pra agenda. **OK.**

### FURO 4 — "não foi 50, foi 45" — Correção de valor

- "não foi 50, foi 45" → Não tem verbo de edição (R6) → Não tem data → Tem valor (45) → R12 manda pra criar_gasto
- Mas o user quer EDITAR, não criar novo gasto
- R2 (retry/falha) pega se a IA acabou de registrar → "não" + valor → retry como criar_gasto
- Mas se não for logo em seguida, R2 não pega

**Mitigação:** R2 já cobre isso quando é no contexto imediato. Fora de contexto, o user provavelmente diria "muda o valor do uber pra 45" que pega em R6.

**STATUS: RISCO BAIXO — R2 mitiga no contexto imediato**

---

## Branch 3: buscar (R5) — 13-buscar-financeiro

### Funciona bem:
- "quanto gastei hoje?" → R5 ✓
- "meus gastos da semana" → R5 ✓

### FURO 5 — "gastos" sem verbo de busca

- "gastos de março" → Tem verbo de busca? "gastos" está na lista? Sim: "meus gastos" → R5 ✓
- "alimentação esse mês" → NÃO tem verbo de busca → NÃO tem valor → NÃO tem data clara → R14 padrao → ERRADO, user quer ver gastos de alimentação

**Mensagens que são busca mas não têm verbo de busca:**
- "alimentação esse mês" → padrao (errado, deveria ser buscar)
- "transporte" → padrao (ambíguo — quer ver gastos? criar?)
- "uber" → R12 pega como gasto? Não, sem valor → padrao

**Mitigação:** Na v2 original isso também não funcionava. O prompt padrao (12-padrao.txt) tem: "Ambíguo → UMA pergunta: 'Quer registrar como gasto ou colocar na agenda?'". Então o user recebe uma pergunta de esclarecimento. **Aceitável.**

**STATUS: RISCO BAIXO — padrao faz pergunta de esclarecimento**

---

## Branch 4: buscar_evento_agenda (R5) — 05-buscar-agenda

### Funciona bem:
- "o que tenho amanhã?" → R5 ✓
- "agenda de sexta" → R5 ✓

### FURO 6 — "sexta" sozinho, sem verbo de busca

- "sexta" → Não tem verbo de busca → R11 pega (tem data) → criar_evento_agenda → ERRADO?

Na verdade, "sexta" sozinho sem mais contexto é ambíguo:
- Pode ser "o que tenho sexta?" (busca)
- Pode ser "agenda aí sexta" (criar, mas criar o quê?)

O prompt 03-criar-evento tem: "Só data sem horário → criar com horário padrão 09:00" e "FALLBACK: 'Compromisso'/'Lembrete' se frase tiver APENAS data/hora sem assunto"

Então "sexta" sozinho criaria um compromisso genérico chamado "Compromisso" na sexta 09:00. Não é ideal. Mas é edge case raro — ninguém manda só "sexta".

**STATUS: RISCO MUITO BAIXO — edge case irrealista**

---

## Branch 5: editar_evento_agenda (R6) — 08-editar-evento

### Funciona bem:
- "muda a reunião pra quinta" → R6 ✓
- "reagenda o almoço pra sexta 12h" → R6 ✓

### FURO 7 — "reunião às 15h ao invés de 14h"

- "reunião às 15h ao invés de 14h" → NÃO tem verbo de edição ("muda", "troca", "altera")
- R11 pega (tem horário) → criar_evento_agenda → ERRADO, user quer editar

Mas quem fala assim? Mais provável: "muda a reunião pra 15h" (R6 pega). Ou no contexto de R2 (retry, logo após criar errado).

**Mitigação:** "ao invés de", "em vez de" poderia ser adicionado como sinal de edição. Mas é edge case raro.

**STATUS: RISCO BAIXO — linguagem incomum, R2 cobre no contexto**

---

### FURO 8 — "coloca a reunião pra mais cedo"

- "coloca a reunião pra mais cedo" → NÃO tem verbo de edição padrão, NÃO tem horário
- R14 padrao? Ou R11 (se interpretar "mais cedo" como referência temporal)?

"mais cedo", "mais tarde", "depois" → não são horários explícitos. Provavelmente cai em padrao, que pergunta esclarecimento. **Aceitável.**

**STATUS: RISCO BAIXO**

---

## Branch 6: editar (R6) — 09-editar-financeiro

### Funciona bem:
- "muda o valor do uber pra 30" → R6 ✓
- "corrige o nome, era mercado" → R6 ✓

### Sem furos identificados. A detecção por verbo ("muda", "troca", "corrige") é robusta.

---

## Branch 7: excluir_evento_agenda (R7) — 06-excluir-evento

### Funciona bem:
- "cancela o dentista de sexta" → R7 ✓
- "não vou no dentista sexta" → R7 ✓

### FURO 9 — "tira o almoço" (sem data)

- "tira o almoço" → R7 pega ("tira" é verbo de exclusão) → excluir_evento_agenda ✓

Mas é agenda ou financeiro? "almoço" pode ser gasto ou evento.
R7 diz: "Se user falar em apagar/deletar/remover/excluir/tirar → Agenda ou Financeiro?"

Na v4, R7 tem sub-decisão:
- Agenda → excluir_evento_agenda
- Financeiro → excluir

Mas como o classificador decide se é agenda ou financeiro? Pela palavra? "almoço" é ambíguo.

Na v2 original, era a mesma situação. O prompt de exclusão (06-excluir-evento) busca por nome na tabela. Se encontrar, exclui. Se não encontrar, diz que não encontrou.

**Mitigação:** Se o user disser "tira o almoço" e existir um EVENTO chamado "Almoço" e um GASTO chamado "Almoço", o classificador vai pra um dos dois e o outro fica intacto. Isso é uma limitação do single-branch, não do classificador.

**STATUS: LIMITAÇÃO ARQUITETURAL — não resolve no classificador**

---

### FURO 10 — "não preciso mais do dentista sexta"

- "não preciso mais do dentista sexta" → R7 tem "não vou mais" como negação
- "não preciso mais" NÃO está na lista de negações
- R11 pega (tem "sexta") → criar_evento_agenda → ERRADO, user quer cancelar

**Adicionar ao R7:** "não preciso mais", "deixa pra lá", "esquece"

**STATUS: FURO REAL — precisa adicionar negações**

---

## Branch 8: excluir (R7) — 07-excluir-financeiro

### Funciona bem:
- "apaga o gasto do uber" → R7 ✓
- "remove o último gasto" → R7 ✓

### Sem furos adicionais. Mesma lógica do evento.

---

## Branch 9: criar_evento_recorrente (R10) — 11-recorrente

### Funciona bem:
- "treino todo dia 7h" → R10 ✓
- "yoga toda terça e quinta 7h" → R10 ✓

### FURO 11 — "me lembra todo dia 5 de pagar aluguel"

- "me lembra todo dia 5 de pagar aluguel" → R10 pega ("todo dia") → criar_evento_recorrente ✓
- Mas "me lembra" sugere lembrete, não evento recorrente
- Na prática, o prompt 11-recorrente cria um evento recorrente que funciona como lembrete → **OK**

**STATUS: SEM FURO — funciona por coincidência**

---

### FURO 12 — "academia segunda quarta e sexta 6h"

- "academia segunda quarta e sexta 6h" → R10 pega? NÃO tem "todo", "toda", "de X em X"
- R11 pega (tem horário) → criar_evento_agenda → cria UM evento na segunda 6h → ERRADO, user quer recorrente nos 3 dias

**Mitigação:** Adicionar ao R10: detecção de múltiplos dias da semana sem "todo/toda" como sinal de recorrência.
"segunda quarta e sexta" / "terça e quinta" → padrão de recorrência implícita.

**STATUS: FURO REAL — recorrência implícita não detectada**

---

## Branch 10: criar_limite (R8)

### Funciona bem. Detecção por palavra-chave "limite" é robusta.
### Sem furos identificados.

---

## Branch 11: relatorio_semanal / relatorio_mensal (R4)

### Funciona bem:
- "relatório da semana" → R4 ✓
- "resumo do mês" → R4 ✓

### FURO 13 — "como foram meus gastos essa semana?"

- "como foram meus gastos essa semana?" → R5 pega? "meus gastos" está na lista de busca → buscar ✓
- Mas o user pode querer o RELATÓRIO (PDF/XLSX), não a listagem no chat
- Na prática, buscar retorna listagem e o user pode pedir relatório se quiser → **Aceitável**

**STATUS: SEM FURO — busca e relatório são coisas diferentes e o user sabe pedir**

---

## Branch 12: padrao (R14) — 12-padrao

### Funciona bem:
- "oi" → R14 ✓
- "como funciona?" → R14 ✓

### FURO 14 — "obrigado" com histórico de criação

- "obrigado" → R14 padrao → responde "De nada!" ✓
- Mas se o user acabou de pedir algo e diz "obrigado" antes da IA confirmar, R1 não pega (não é "sim")
- Não é furo, é comportamento esperado. "Obrigado" é conversa genérica.

**STATUS: SEM FURO**

---

## Branch 13: Receitas (R3)

### FURO 15 — "recebi o relatório" (falso positivo)

- "recebi o relatório" → R3 pega? "recebi" + número? Não tem número → R3 não pega → R14 padrao ✓
- "recebi o email das 10h" → R3? "recebi" + "10"? 10 é horário, não valor
- R3 exige "recebi" + QUALQUER número → "10" poderia ser interpretado como valor

**Mas o prompt v2 original também tem esse risco.** O prompt 04-registrar-gasto tem:
> "Se houver múltiplos números: priorize o que tiver aparência monetária. Ignore horários (8:30) e datas (12/01)."

Então "recebi o email das 10h" → R3 manda pra criar_gasto → o prompt de gasto vê "10" mas ignora porque parece horário (10h). Resultado: acao=padrao ou registro errado de R$10.

**Mitigação possível:** R3 deveria exigir número que NÃO seja seguido de "h", ":", "min". Mas isso complica a regra.

**STATUS: RISCO BAIXO — edge case raro, prompt de gasto tem filtro parcial**

---

## FURO 16 — "buscar crianças amanhã 11h30" — conflito R5 vs R11

- "buscar crianças amanhã 11h30" → R5 checa verbo de busca: "buscar" ESTÁ na lista
- R5 mandaria pra buscar_evento_agenda → ERRADO, user quer AGENDAR

O verbo "buscar" tem significado duplo:
- "buscar meus gastos" → consultar (busca financeira)
- "buscar as crianças 11h30" → ação física (agenda)

R5 está ANTES de R11 na ordem de prioridade. Então "buscar" + data + não é financeiro → R5 manda pra buscar_evento_agenda.

**Isso é um FURO GRAVE.** O classificador confundiria "buscar crianças amanhã 11h30" com busca de eventos.

**Mitigação:**
R5 deveria checar: verbo de busca + CONTEXTO FINANCEIRO → buscar financeiro. Verbo de busca + CONTEXTO AGENDA → buscar_evento_agenda. Verbo de busca + SEM CONTEXTO DE CONSULTA → NÃO é busca, é ação.

Ou melhor: R5 deveria exigir que o verbo de busca esteja em CONTEXTO DE CONSULTA:
- "buscar meus gastos" → consulta (buscar financeiro)
- "buscar crianças 11h30" → ação (criar agenda)

Diferença: "buscar" + {dados/registros/gastos/agenda/eventos/compromissos} = CONSULTA
"buscar" + {pessoa/objeto/lugar} + horário = AÇÃO FÍSICA = AGENDA

**STATUS: FURO GRAVE — "buscar" como verbo de busca captura ações físicas**

---

## Resumo de Furos

| # | Furo | Severidade | Status |
|---|------|-----------|--------|
| 1 | Branch `criar_lembrete` pode existir no N8N mas não no classificador | VERIFICAR | Precisa checar no N8N |
| 2 | Dois prompts de criar evento (03 e prompt_criar1) — qual é o ativo? | VERIFICAR | Precisa checar no N8N |
| 3 | ~~"paguei amanhã"~~ — não acontece na prática | NENHUM | OK |
| 4 | "não foi 50, foi 45" — pode criar novo ao invés de editar | BAIXO | R2 mitiga |
| 5 | "alimentação esse mês" — busca sem verbo de busca | BAIXO | padrao pergunta |
| 6 | "sexta" sozinho → cria compromisso genérico | MUITO BAIXO | Edge case raro |
| 7 | "reunião às 15h ao invés de 14h" — edição sem verbo de edição | BAIXO | R2 mitiga |
| 8 | "coloca a reunião pra mais cedo" — sem horário concreto | BAIXO | padrao pergunta |
| 9 | "tira o almoço" — ambíguo agenda vs financeiro | ARQUITETURAL | Limitação single-branch |
| 10 | "não preciso mais", "deixa pra lá", "esquece" — negações não cobertas | **MEDIO** | **Adicionar ao R7** |
| 11 | ~~"me lembra todo dia 5"~~ — funciona via R10 | NENHUM | OK |
| 12 | "academia segunda quarta e sexta 6h" — recorrência implícita | **MEDIO** | **Adicionar ao R10** |
| 13 | ~~"como foram meus gastos?"~~ — busca funciona | NENHUM | OK |
| 14 | ~~"obrigado"~~ — padrao funciona | NENHUM | OK |
| 15 | "recebi o email das 10h" — falso positivo R3 | BAIXO | Prompt filtra parcialmente |
| 16 | "buscar crianças 11h30" — "buscar" confunde com busca | **GRAVE** | **Reescrever R5** |

---

## Furos que precisam de correção na v4

### FURO 10 — Negações não cobertas no R7

**Adicionar ao R7:**
```
Negações adicionais: "não preciso mais", "deixa pra lá", "esquece",
"esquece o/a", "não quero mais", "tanto faz", "deixa quieto",
"não vai rolar", "mudou", "caiu" (no sentido de cancelamento)
```

### FURO 12 — Recorrência implícita não detectada no R10

**Adicionar ao R10:**
```
Recorrência IMPLÍCITA (sem "todo/toda"):
Se a mensagem listar 2+ dias da semana + horário → criar_evento_recorrente.
"academia segunda quarta e sexta 6h" → FREQ=WEEKLY;BYDAY=MO,WE,FR
"inglês terça e quinta 19h" → FREQ=WEEKLY;BYDAY=TU,TH
"pilates segunda e quarta 7h" → FREQ=WEEKLY;BYDAY=MO,WE
```

### FURO 16 — "buscar" como verbo de busca captura ações físicas

**Reescrever R5:**
```
## R5 — BUSCA EXPLÍCITA
Verbos de busca: "quanto gastei", "meus gastos", "me mostra", "lista",
"quais", "o que tem", "o que tenho", "agenda de", "quero ver",
"como estão", "resumo"

ATENÇÃO: "buscar" e "procurar" são AMBÍGUOS:
- "buscar meus gastos" → CONSULTA → buscar (financeiro)
- "buscar crianças 11h30" → AÇÃO FÍSICA → NÃO é busca, pular para R11 (agenda)

REGRA: "buscar/procurar" só é BUSCA se seguido de:
  dados, registros, gastos, eventos, agenda, compromissos, relatório,
  informações, histórico, extrato
Se seguido de pessoa, objeto ou lugar + horário → NÃO é busca → pular para próxima regra.
```
