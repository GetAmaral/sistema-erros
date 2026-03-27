# Prompt Final — prompt_criar1 (v3)

**Data:** 2026-03-27
**Versao:** 3.0
**Mudancas:** Remocao da rejeicao + regra de confianca + dica educativa + valor no nome do evento

---

## Trecho a substituir

Substituir APENAS as primeiras linhas do prompt_criar1. O restante (REGRA ZERO, FORMATO JSON, DECISÃO criar_evento vs padrao, compromisso_tipo, DURAÇÃO, CONTEXTO DE TEMPO, CALENDÁRIO DINÂMICO, RESOLUÇÃO DE DATAS, PARSING DE EVENTOS, NOME DO EVENTO, LEMBRETES, EMOJIS, REGRAS FINAIS) permanece INALTERADO.

---

### ANTES (linhas 1-2 atuais):

```
==Módulo AGENDA — CRIAR EVENTOS. Tudo é EVENTO: avisos, alertas, lembretes, compromissos. Diferença = TIPO (compromisso/lembrete) e DURAÇÃO.
Fora de agenda (finanças, compras, preços) → acao="padrao", tool=[], mensagem="Essa mensagem não parece ser sobre agenda."
```

### DEPOIS (substituir por):

```
==Módulo AGENDA — CRIAR EVENTOS. Tudo é EVENTO: avisos, alertas, lembretes, compromissos. Diferença = TIPO (compromisso/lembrete) e DURAÇÃO.

REGRA DE CONFIANÇA: Se você recebeu esta mensagem, o Classificador JÁ decidiu que é agenda. NÃO questione. Seu trabalho é EXTRAIR nome + data/hora e CRIAR o evento.

PROIBIDO: A frase "Essa mensagem não parece ser sobre agenda." está BANIDA. Nunca use.

VERBOS INCOMUNS: O verbo do user pode ser qualquer coisa ("enviar mensagem", "pagar", "comprar", "mandar", "ligar", "falar"). Isso NÃO significa que está fora de agenda. O verbo + complemento vira o NOME do evento:
  "enviar mensagem Gustavo Garcia as 10h30" → nome="Enviar Mensagem Gustavo Garcia"
  "ligar pro João às 15h" → nome="Ligar pro João"
  "pagar 100 reais amanhã pro Luan" → nome="Pagar R$100 pro Luan"
  "comprar presente de 100 sábado" → nome="Comprar Presente (R$100)"
  "levo o carro na oficina 14h 300 reais" → nome="Levar Carro na Oficina (R$300)"
  "buscar as crianças 11h30" → nome="Buscar as Crianças"
  "falar com Maria 9h" → nome="Falar com Maria"
  "passar no mercado 18h" → nome="Passar no Mercado"

VALOR NA MENSAGEM: Se a mensagem contiver valor (R$, reais), o valor entra no NOME do evento entre parênteses. O valor NÃO é registrado como gasto — isso é responsabilidade de outro módulo.

Se GENUINAMENTE não conseguir extrair nome NEM data/hora:
→ acao="padrao", tool=[], mensagem="Não consegui identificar o evento. Pode me dizer o que e quando?"

DICA EDUCATIVA (adicionar ao final da mensagem de sucesso):
Mostrar a dica SOMENTE quando a mensagem original se encaixar em um destes casos:

CASO 1 — Verbo de comunicação + horário:
  Gatilho: "enviar", "mandar", "avisar", "falar" + horário
  Dica: "\n\n💡 Para aproveitar o máximo do Total Assistente, tente: 'reunião com Gustavo amanhã 10h30' ou 'me lembra de pagar o boleto dia 30'"

CASO 2 — Valor monetário presente na mensagem:
  Gatilho: "reais", "R$" ou número que claramente é dinheiro
  Dica: "\n\n💡 Notei que você mencionou R$[valor]. Depois que [verbo no passado], me manda: '[nome] [valor] reais' pra eu registrar o gasto."
  Exemplos:
    "pagar 100 amanhã pro Luan" → "💡 Notei que você mencionou R$100. Depois que pagar, me manda: 'paguei 100 pro Luan' pra eu registrar o gasto."
    "café com Ana 15 reais segunda" → "💡 Notei que você mencionou R$15. Depois do café, me manda: 'café 15 reais' pra eu registrar o gasto."
    "comprar presente de 100 sábado" → "💡 Notei que você mencionou R$100. Depois que comprar, me manda: 'presente 100 reais' pra eu registrar o gasto."

CASO 3 — Nome muito vago:
  Gatilho: nome ficou genérico tipo "Compromisso" ou "Lembrete" sem especificidade
  Dica: "\n\n💡 Para eu organizar melhor sua agenda, tente incluir o assunto: 'reunião com João 14h' ou 'dentista sexta 10h'"

NÃO MOSTRAR DICA quando:
  - Mensagem clara e direta ("reunião com Luan 16h", "dentista sexta 14h")
  - Lembrete explícito ("me lembra de tomar remédio 9h")
  - User já recebeu dica nas últimas 4 mensagens (verificar histórico)
  - Nunca 2 dicas seguidas

FREQUÊNCIA: Máximo 1 dica a cada 5 mensagens. Na dúvida, NÃO mostre.
```

---

## Exemplos de output completo (antes vs depois)

### Exemplo 1 — Verbo de comunicação

**ANTES:**
```json
{ "acao": "padrao", "tool": [], "mensagem": "Essa mensagem não parece ser sobre agenda." }
```

**DEPOIS:**
```json
{
  "acao": "criar_evento",
  "tool": [{
    "nome_evento": "Enviar Mensagem Gustavo Garcia",
    "descricao_evento": "",
    "data_inicio_evento": "2026-03-27T10:30:00-03:00",
    "data_fim_evento": "2026-03-27T10:45:00-03:00",
    "compromisso_tipo": "lembrete"
  }],
  "mensagem": "✅ Evento agendado!\n📅 Enviar Mensagem Gustavo Garcia ⏰ hoje às 10:30\n\n💡 Para aproveitar o máximo do Total Assistente, tente: 'reunião com Gustavo amanhã 10h30' ou 'me lembra de pagar o boleto dia 30'"
}
```

### Exemplo 2 — Valor + data (lembrete de pagamento)

**Input:** "pagar 100 reais amanhã pro Luan"

**ANTES:** Dependia do humor do classificador. Às vezes gasto, às vezes agenda, às vezes padrão.

**DEPOIS:**
```json
{
  "acao": "criar_evento",
  "tool": [{
    "nome_evento": "Pagar R$100 pro Luan",
    "descricao_evento": "Valor: R$100",
    "data_inicio_evento": "2026-03-28T09:00:00-03:00",
    "data_fim_evento": "2026-03-28T09:15:00-03:00",
    "compromisso_tipo": "lembrete"
  }],
  "mensagem": "✅ Evento agendado!\n📅 Pagar R$100 pro Luan ⏰ amanhã às 09:00\n\n💡 Notei que você mencionou R$100. Depois que pagar, me manda: 'paguei 100 pro Luan' pra eu registrar o gasto."
}
```

### Exemplo 3 — Mensagem clara (sem dica)

**Input:** "reunião com Luan amanhã 16h"

**DEPOIS:**
```json
{
  "acao": "criar_evento",
  "tool": [{
    "nome_evento": "Reunião com Luan",
    "descricao_evento": "",
    "data_inicio_evento": "2026-03-28T16:00:00-03:00",
    "data_fim_evento": "2026-03-28T16:30:00-03:00",
    "compromisso_tipo": "compromisso"
  }],
  "mensagem": "✅ Evento agendado!\n📅 Reunião com Luan ⏰ amanhã às 16:00"
}
```

### Exemplo 4 — Zona cinzenta (café + valor + dia)

**Input:** "café com a Ana 15 reais segunda"

**DEPOIS:**
```json
{
  "acao": "criar_evento",
  "tool": [{
    "nome_evento": "Café com Ana (R$15)",
    "descricao_evento": "Valor estimado: R$15",
    "data_inicio_evento": "2026-03-30T09:00:00-03:00",
    "data_fim_evento": "2026-03-30T09:30:00-03:00",
    "compromisso_tipo": "compromisso"
  }],
  "mensagem": "✅ Evento agendado!\n📅 Café com Ana (R$15) ⏰ segunda às 09:00\n\n💡 Notei que você mencionou R$15. Depois do café, me manda: 'café 15 reais' pra eu registrar o gasto."
}
```

### Exemplo 5 — Presente tense sem data (gasto, nao chega aqui)

**Input:** "pago 100 pro Luan" → NÃO chega no prompt_criar1, o classificador manda pra criar_gasto.

### Exemplo 6 — Presente tense com data (chega aqui)

**Input:** "pago 100 pro Luan amanhã"

**DEPOIS:**
```json
{
  "acao": "criar_evento",
  "tool": [{
    "nome_evento": "Pagar R$100 pro Luan",
    "descricao_evento": "Valor: R$100",
    "data_inicio_evento": "2026-03-28T09:00:00-03:00",
    "data_fim_evento": "2026-03-28T09:15:00-03:00",
    "compromisso_tipo": "lembrete"
  }],
  "mensagem": "✅ Evento agendado!\n📅 Pagar R$100 pro Luan ⏰ amanhã às 09:00\n\n💡 Notei que você mencionou R$100. Depois que pagar, me manda: 'paguei 100 pro Luan' pra eu registrar o gasto."
}
```
