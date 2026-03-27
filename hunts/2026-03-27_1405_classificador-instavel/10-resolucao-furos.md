# Resolucao de Furos — Analise Final

**Data:** 2026-03-27

---

## Furo 1 — Branch `criar_lembrete_agenda` / `prompt_lembrete`

### Diagnostico

- O N8N tem um Set node `prompt_lembrete` com o conteudo de `10-lembrete.txt`
- O classificador NUNCA teve `criar_lembrete_agenda` como branch
- O prompt `prompt_criar1` (03-criar-evento.txt) ja engloba lembretes via `compromisso_tipo: "lembrete"`
- O node `prompt_lembrete` e **dead code** — nunca e atingido

### Resolucao

- **Classificador v4:** NAO adicionar `criar_lembrete_agenda`. Manter como esta.
- **N8N:** Usuario vai remover o node `prompt_lembrete` e o branch correspondente no Switch.
- **Prompt 10-lembrete.txt:** Marcar como DEPRECADO. Funcionalidade absorvida pelo `prompt_criar1`.

**STATUS: RESOLVIDO — dead code a ser removido pelo usuario no N8N**

---

## Furo 2 — `gerar_relatorio` vs `relatorio_semanal` / `relatorio_mensal`

### Diagnostico

Conferi o Guia de Migracao (00-GUIA-MIGRAÇÃO.md):

```
14-relatorio-semanal.txt → Set node "prompt_rel_semanal"
15-relatorio-mensal.txt  → Set node "prompt_rel_mensal"
```

O Switch do N8N tem 2 outputs separados para relatorio, cada um apontando para seu Set node. Os nomes no classificador (`relatorio_semanal` / `relatorio_mensal`) correspondem aos outputs do Switch.

### Resolucao

**NAO e furo.** Os nomes batem. O classificador v4 esta correto com `relatorio_semanal` e `relatorio_mensal` como branches separados.

**STATUS: RESOLVIDO — sem furo**

---

## Mapa definitivo: Classificador v4 vs N8N Switch

| Branch no Classificador | Output do Switch | Set Node | Prompt |
|------------------------|-----------------|----------|--------|
| `criar_gasto` | ✓ | `registrar_gasto` | 04-registrar-gasto.txt |
| `buscar` | ✓ | `buscar_gasto` | 13-buscar-financeiro.txt |
| `editar` | ✓ | `editar_gasto` | 09-editar-financeiro.txt |
| `excluir` | ✓ | `excluir2` | 07-excluir-financeiro.txt |
| `criar_evento_agenda` | ✓ | `prompt_criar1` | 03-criar-evento.txt (inclui lembretes) |
| `buscar_evento_agenda` | ✓ | `prompt_busca1` | 05-buscar-agenda.txt |
| `editar_evento_agenda` | ✓ | `prompt_editar1` | 08-editar-evento.txt |
| `excluir_evento_agenda` | ✓ | `prompt_excluir` | 06-excluir-evento.txt |
| `criar_evento_recorrente` | ✓ | `prompt_lembrete1` | 11-recorrente.txt |
| `relatorio_semanal` | ✓ | `prompt_rel_semanal` | 14-relatorio-semanal.txt |
| `relatorio_mensal` | ✓ | `prompt_rel_mensal` | 15-relatorio-mensal.txt |
| `padrao` | ✓ | `padrao` | 12-padrao.txt |
| `criar_limite` | ✓ | ? | Nao mapeado no guia |
| `editar_limite` | ✓ | ? | Nao mapeado no guia |
| `excluir_limite` | ✓ | ? | Nao mapeado no guia |
| — | DEAD CODE | `prompt_lembrete` | 10-lembrete.txt (DEPRECADO) |

### Acao no N8N (para o usuario)

1. Remover o node `prompt_lembrete` do workflow Fix Conflito v2
2. Remover o output correspondente do Switch (se existir `criar_lembrete_agenda`)
3. Confirmar que `prompt_criar1` trata lembretes corretamente (ja confirmado pelo prompt)

---

## Furos restantes da analise — Correções para v4.1

### Furo 10 — Negacoes nao cobertas (R7)

**Adicionar ao R7:**
```
Negações adicionais que indicam CANCELAMENTO:
"não preciso mais", "deixa pra lá", "esquece", "esquece o/a",
"não quero mais", "não vai rolar", "deixa quieto", "mudou de ideia"
```

### Furo 12 — Recorrencia implicita (R10)

**Adicionar ao R10:**
```
Recorrência IMPLÍCITA (sem "todo/toda"):
Se a mensagem listar 2+ dias da semana + horário → criar_evento_recorrente.
"academia segunda quarta e sexta 6h" → criar_evento_recorrente
"inglês terça e quinta 19h" → criar_evento_recorrente
```

### Furo 16 — "buscar" ambiguo (R5)

**Reescrever no R5:**
```
ATENÇÃO com "buscar" e "procurar" — são AMBÍGUOS:
"buscar meus gastos" → CONSULTA → buscar
"buscar crianças 11h30" → AÇÃO FÍSICA → NÃO é busca → pular para R11

REGRA: "buscar/procurar" só é busca se seguido de:
  dados, registros, gastos, eventos, agenda, compromissos,
  informações, histórico, extrato, relatório
Se seguido de pessoa/objeto/lugar + horário → pular para próxima regra.
```
