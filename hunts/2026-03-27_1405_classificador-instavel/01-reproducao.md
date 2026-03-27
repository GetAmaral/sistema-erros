# Fase 1 — Reproducao

**Data:** 2026-03-27
**Metodo:** 5 envios da mesma mensagem via webhook DEV, com limpeza automatica de contexto (~10s)

## Teste Piloto — Mensagem Original

**Mensagem:** `Enviar mensagem Gustavo Garcia as 10h30`

| # | Horario UTC | Resposta da IA | Veredicto |
|---|-------------|----------------|-----------|
| 1 | 14:05:38 | "Essa mensagem nao parece ser sobre agenda." | FAIL |
| 2 | 14:05:58 | "Essa mensagem nao parece ser sobre agenda." | FAIL |
| 3 | 14:06:18 | "Evento agendado! Enviar Mensagem a Gustavo Garcia hoje as 10h30" | PASS |
| 4 | 14:06:35 | "Essa mensagem nao parece ser sobre agenda." | FAIL |
| 5 | 14:07:08 | "Evento agendado! Enviar Mensagem Gustavo Garcia hoje as 10:30" | PASS |

**Taxa de reproducao do bug:** 3/5 (60%)
**Classificacao:** FREQUENTE

## Conclusao

Bug reproduzido com taxa de 60%. Mesma mensagem, mesmo user, contexto limpo — resultado diferente. Instabilidade confirmada no classificador.
