# Hunt: Classificador Instavel com Verbos Ambiguos

**Inicio:** 2026-03-27 14:05 UTC
**Reportado por:** Luiz Felipe (usuario)

## O que foi reportado

Ao enviar "Enviar mensagem Gustavo Garcia as 10h30" para o bot, a resposta e inconsistente — as vezes agenda o evento, as vezes rejeita dizendo "nao parece ser sobre agenda".

## Mensagem/Acao

`Enviar mensagem Gustavo Garcia as 10h30`

## Esperado vs Observado

- **Esperado:** IA deveria sempre criar evento "Enviar mensagem Gustavo Garcia" as 10h30
- **Observado:** 3/5 vezes rejeita como "nao parece ser sobre agenda", 2/5 vezes agenda corretamente

## Hipotese inicial

A palavra "mensagem" no input confunde o Classificador (Escolher Branch), que interpreta como conversa generica ao inves de agenda. O horario "10h30" deveria ser sinal forte de agenda, mas a palavra "mensagem" compete e vence em 60% dos casos.

## Funcionalidades possivelmente afetadas

- [x] bot-whatsapp/01-roteador-principal (Escolher Branch)
- [x] agenda/01-agendamento-proprio (criacao de evento)
- [ ] bot-whatsapp/07-conversa-generica (branch padrao captura indevidamente)
- [ ] bot-whatsapp/09-multi-intencao (mensagens com sinais conflitantes)
