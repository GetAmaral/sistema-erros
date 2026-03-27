# Analise — IA chamando tools indevidas durante criacao

**Data:** 2026-03-27

---

## O Problema

O user relata que ao criar um evento, a IA as vezes:
- Chama a tool de busca (faz uma conferencia no banco antes de criar)
- Chama a tool de relatorio e manda um relatorio pro user
- Comportamento inconsistente e estranho

## Causa Raiz

### Arquitetura do AI Agent no N8N

```
Switch - Branches1
  ├── criar_evento_agenda → Set node "prompt_criar1" ─┐
  ├── buscar_evento_agenda → Set node "prompt_busca1" ─┤
  ├── excluir_evento_agenda → Set node "prompt_excluir" ─┤
  ├── criar_gasto → Set node "registrar_gasto" ─┤
  ├── gerar_relatorio → Set node "prompt_rel" ─┤
  └── padrao → Set node "padrao" ─┤
                                                        ↓
                                              Aggregate (junta prompt)
                                                        ↓
                                              AI Agent (GPT-4.1-mini)
                                              ├── Tool: buscar_eventos
                                              ├── Tool: excluir_evento
                                              ├── Tool: editar_eventos
                                              ├── Tool: buscar_financeiro
                                              ├── Tool: editar_financeiro
                                              ├── Tool: excluir_financeiro
                                              ├── Tool: relatorios_semanais
                                              ├── Tool: relatorios_mensais
                                              └── (todas as outras tools)
```

**O AI Agent é UM SÓ node** com **TODAS as tools conectadas**. O que muda entre branches é apenas o prompt injetado. Mas as tools ficam sempre acessíveis.

### O conflito no master-system.txt

```
# MODOS DE OPERAÇÃO
(a) MODO JSON/BUILDER → monte JSON, NÃO chame tools.    ← prompt_criar1 diz isso
(b) MODO TOOL/EXECUTOR → chame a tool indicada           ← prompt_busca1 diz isso

# REGRAS DE TOOLS
- Na dúvida entre "chutar" e chamar tool → chame a tool. ← CONTRADIZ o modo (a)
```

O prompt_criar1 diz "NÃO chame tools", mas o master-system diz "na dúvida, chame tool".
Quando a IA recebe uma mensagem ambígua no branch criar_evento_agenda, ela pode:
1. Seguir o prompt_criar1 → montar JSON sem chamar nada ✓
2. Seguir o master-system → "na dúvida, chame tool" → chama buscar_eventos ou relatorios_semanais ✗

### Por que chama relatório?

O AI Agent vê TODAS as tools disponíveis, incluindo `relatorios_semanais`. Se a mensagem do user contiver algo que remotamente soe como relatório (ex: "me mostra", "resumo"), a IA pode decidir chamar essa tool mesmo estando no branch de criação de evento.

### Por que faz busca antes de criar?

A IA "quer ser útil" — antes de criar um evento, decide verificar se já existe um igual. Isso é um comportamento emergente do LLM que tem acesso a `buscar_eventos`. Não foi instruído a fazer isso, mas tem a tool disponível e o instinto de "verificar antes".

---

## Solução

### Opção A — Corrigir no master-system.txt (rápido, sem mudar N8N)

Mudar a regra de tools no master-system para respeitar o modo do prompt:

**ANTES:**
```
# REGRAS DE TOOLS
- Chame a tool de verdade ou não chame. Nunca finja que chamou.
- Máximo 1 chamada por tool por turno (exceto quando processar lista de múltiplos itens).
- Use parâmetros exatamente no formato do PROMPT ESPECÍFICO.
- Baseie a resposta final na resposta da tool, sem inventar dados.
- Na dúvida entre "chutar" e chamar tool → chame a tool.
```

**DEPOIS:**
```
# REGRAS DE TOOLS
- REGRA MÁXIMA: Se o PROMPT ESPECÍFICO disser "NÃO chame tools" ou "Apenas monte JSON" → você NÃO pode chamar NENHUMA tool. Nenhuma. Zero. Nem buscar, nem editar, nem relatório. Apenas monte o JSON e retorne.
- Se o PROMPT ESPECÍFICO disser "DEVE chamar a tool X" → chame APENAS a tool X. Nenhuma outra.
- Chame a tool de verdade ou não chame. Nunca finja que chamou.
- Máximo 1 chamada por tool por turno (exceto quando processar lista de múltiplos itens).
- Use parâmetros exatamente no formato do PROMPT ESPECÍFICO.
- Baseie a resposta final na resposta da tool, sem inventar dados.
- Na dúvida entre "chutar" e chamar tool → SOMENTE se o PROMPT ESPECÍFICO autoriza tools. Se ele disser "NÃO chame tools", a resposta é SEMPRE "não chame".
```

### Opção B — Isolar tools por branch no N8N (ideal, mas mais trabalho)

Criar AI Agents separados por grupo de funcionalidade:
- AI Agent Criador (tools: nenhuma — só monta JSON)
- AI Agent Buscador (tools: buscar_eventos, buscar_financeiro)
- AI Agent Editor (tools: buscar + editar)
- AI Agent Excluidor (tools: buscar + excluir)
- AI Agent Relatório (tools: relatorios_semanais, relatorios_mensais)

Isso eliminaria o problema na raiz — a IA não pode chamar o que não tem. Mas exige reestruturar o workflow.

### Opção C — Reforço no prompt_criar1 (complementar à A)

Adicionar no prompt_criar1:

```
PROIBIDO CHAMAR TOOLS: Você está no modo JSON/BUILDER.
NÃO chame buscar_eventos. NÃO chame relatorios_semanais. NÃO chame relatorios_mensais.
NÃO chame editar_eventos. NÃO chame excluir_evento. NÃO chame buscar_financeiro.
NÃO chame NENHUMA tool. APENAS retorne o JSON.
Se você chamar qualquer tool, o sistema vai quebrar.
```

Mesmo reforço nos outros prompts de modo JSON/BUILDER:
- `04-registrar-gasto.txt` (já tem "NÃO chama tools")
- `10-lembrete.txt` (já tem "NÃO chama tools")

---

## Recomendação

**Aplicar A + C juntas.** É rápido (só texto) e resolve o problema:

1. **master-system.txt**: Mudar regra de tools para respeitar o modo do prompt
2. **prompt_criar1**: Adicionar lista explícita de tools proibidas
3. **04-registrar-gasto.txt**: Mesmo reforço
4. **12-padrao.txt**: Mesmo reforço

A Opção B (AI Agents separados) é a solução ideal de longo prazo, mas exige refatorar o workflow.

---

## Prompts que precisam de reforço anti-tools

| Prompt | Modo atual | Tools que pode chamar indevidamente |
|--------|-----------|-------------------------------------|
| prompt_criar1 (03) | "NÃO chama tools" | buscar_eventos, relatorios_semanais |
| registrar_gasto (04) | "NÃO chama tools" | buscar_financeiro, relatorios |
| lembrete (10) | "NÃO chama tools" | buscar_eventos |
| padrao (12) | "NÃO executa ações" | qualquer tool |

Os prompts de modo TOOL/EXECUTOR (buscar, editar, excluir) também precisam de reforço, mas ao contrário — devem chamar APENAS a tool indicada:

| Prompt | Deve chamar | NÃO deve chamar |
|--------|-----------|-----------------|
| buscar_agenda (05) | buscar_eventos | relatorios, editar, excluir |
| buscar_financeiro (13) | buscar_financeiro | relatorios, editar_eventos |
| editar_evento (08) | editar_eventos | relatorios, buscar_financeiro |
| excluir_evento (06) | buscar_eventos + excluir_evento | relatorios, editar |
| relatorio_semanal (14) | relatorios_semanais | buscar, editar, excluir |
| relatorio_mensal (15) | relatorios_mensais | buscar, editar, excluir |
