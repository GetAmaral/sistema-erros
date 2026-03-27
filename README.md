# Sistema de Erros — Total Assistente

Repositorio de rastreamento de bugs e erros encontrados no Total Assistente.

Gerado pelo squad **auditor-360 (Lupa)** durante auditorias e hunts no ambiente DEV.

---

## Estrutura

```
hunts/
  └── {YYYY-MM-DD}_{HHmm}_{nome-resumido}/
      ├── 00-contexto.md          # O que foi reportado
      ├── 01-reproducao.md        # Testes de reproducao
      ├── 02-diagnostico.md       # Causa raiz (5 camadas)
      ├── 03-irradiacao.md        # Impacto em outras funcionalidades
      ├── 04-relatorio-final.md   # Consolidacao
      └── dados/                  # Evidencias brutas (JSON)
```

## Hunts

| Data | Slug | Bug | Severidade | Status |
|------|------|-----|-----------|--------|
| 2026-03-27 | classificador-instavel | Classificador oscila em mensagens com verbos ambiguos | ALTA | Corrigido (v4.1) |

## Prompts para Colar no N8N

Pasta `prompts-para-colar/` contém os prompts finais prontos para copiar e colar:

| Arquivo | Node N8N | Ação |
|---------|----------|------|
| `00-master-system-v2.txt` | AI Agent (systemMessage) | Substituir prompt inteiro |
| `01-classificador-v4.1.txt` | Escolher Branch (chainLlm prompt) | Substituir prompt inteiro |
| `02-prompt-criar1-v3.txt` | prompt_criar1 (Set node, campo assignments.prompt.value) | Substituir prompt inteiro |

## Comandos do Squad

- `*hunt "descricao"` — Inicia caca de bug
- `*hunt-status` — Ver hunts em andamento
- `*hunt-list` — Listar todos os hunts
- `*hunt-resume {pasta}` — Retomar hunt
