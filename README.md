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
| 2026-03-27 | classificador-instavel | Classificador oscila em mensagens com verbos ambiguos | ALTA | Documentado |

## Comandos do Squad

- `*hunt "descricao"` — Inicia caca de bug
- `*hunt-status` — Ver hunts em andamento
- `*hunt-list` — Listar todos os hunts
- `*hunt-resume {pasta}` — Retomar hunt
