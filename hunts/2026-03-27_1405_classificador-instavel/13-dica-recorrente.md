# Dica de Recorrência — Adição ao prompt_criar1

**Data:** 2026-03-27

---

## O que adicionar

Um novo CASO 4 na seção de DICA EDUCATIVA do prompt_criar1.

---

## Trecho a adicionar (após CASO 3 da dica educativa)

```
CASO 4 — Múltiplos dias da semana com mesmo assunto (possível recorrência):
  Gatilho: mensagem menciona 2+ dias da semana E o nome extraído é O MESMO
  para todos os eventos criados (ex: "academia segunda quarta e sexta")
  Ação: criar os eventos separadamente (1 por dia, horário padrão) + dica
  Dica: "\n\n💡 Se [nome] se repete toda semana, me manda:
  '[nome] toda segunda quarta e sexta [horário]' que eu crio como recorrente."

  NÃO mostrar esta dica se os eventos têm nomes DIFERENTES.
  "viajo segunda e volto sexta" → 2 eventos diferentes → SEM dica de recorrência.
  "academia segunda quarta e sexta" → mesmo nome → COM dica.
```

---

## Validação — cenários testados

| # | Mensagem | Chega no prompt_criar1? | Eventos criados | Dica? | Correto? |
|---|----------|----------------------|-----------------|-------|----------|
| 1 | "academia segunda quarta e sexta 6h" | NÃO (R10 pega) | — | — | ✓ |
| 2 | "academia toda segunda quarta e sexta 6h" | NÃO (R10 pega) | — | — | ✓ |
| 3 | "academia segunda quarta e sexta" | SIM (R11, sem horário) | 3 (seg 09:00, qua 09:00, sex 09:00) | SIM (mesmo nome) | ✓ |
| 4 | "reunião segunda e terça 14h" | NÃO (R10 pega) | — | — | ✓ |
| 5 | "reunião segunda e terça" | SIM (R11, sem horário) | 2 (seg 09:00, ter 09:00) | SIM (mesmo nome) | ✓ |
| 6 | "viajo segunda e volto sexta" | SIM (R11) | 2 (Viajo seg, Volto sex) | NÃO (nomes diferentes) | ✓ |
| 7 | "dentista segunda e médico quarta" | SIM (R11) | 2 (Dentista seg, Médico qua) | NÃO (nomes diferentes) | ✓ |
| 8 | "academia segunda 6h" | NÃO chega em R10 (1 dia só) | 1 via R11 | NÃO (1 evento só) | ✓ |
| 9 | "academia toda segunda 6h" | NÃO (R10, "toda") | — | — | ✓ |

**Todos os 9 cenários estão corretos.** A dica só aparece quando:
- Mensagem criou 2+ eventos
- Todos com o MESMO nome
- Nenhuma regra anterior (R10) capturou como recorrente

---

## Exemplo de output

**Input:** "academia segunda quarta e sexta"

```json
{
  "acao": "criar_evento",
  "tool": [
    {
      "nome_evento": "Academia",
      "descricao_evento": "",
      "data_inicio_evento": "2026-03-30T09:00:00-03:00",
      "data_fim_evento": "2026-03-30T09:30:00-03:00",
      "compromisso_tipo": "compromisso"
    },
    {
      "nome_evento": "Academia",
      "descricao_evento": "",
      "data_inicio_evento": "2026-04-01T09:00:00-03:00",
      "data_fim_evento": "2026-04-01T09:30:00-03:00",
      "compromisso_tipo": "compromisso"
    },
    {
      "nome_evento": "Academia",
      "descricao_evento": "",
      "data_inicio_evento": "2026-04-03T09:00:00-03:00",
      "data_fim_evento": "2026-04-03T09:30:00-03:00",
      "compromisso_tipo": "compromisso"
    }
  ],
  "mensagem": "✅ Eventos agendados!\n📅 Academia ⏰ segunda às 09:00\n📅 Academia ⏰ quarta às 09:00\n📅 Academia ⏰ sexta às 09:00\n\n💡 Se Academia se repete toda semana, me manda: 'academia toda segunda quarta e sexta 6h' que eu crio como recorrente."
}
```

**Input:** "viajo segunda e volto sexta"

```json
{
  "acao": "criar_evento",
  "tool": [
    {
      "nome_evento": "Viagem",
      "descricao_evento": "",
      "data_inicio_evento": "2026-03-30T09:00:00-03:00",
      "data_fim_evento": "2026-03-30T09:30:00-03:00",
      "compromisso_tipo": "compromisso"
    },
    {
      "nome_evento": "Volta da Viagem",
      "descricao_evento": "",
      "data_inicio_evento": "2026-04-03T09:00:00-03:00",
      "data_fim_evento": "2026-04-03T09:30:00-03:00",
      "compromisso_tipo": "compromisso"
    }
  ],
  "mensagem": "✅ Eventos agendados!\n📅 Viagem ⏰ segunda às 09:00\n📅 Volta da Viagem ⏰ sexta às 09:00"
}
```

Sem dica, porque os nomes são diferentes.
