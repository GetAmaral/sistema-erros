# Investigação — Testes X1, A2, R1

3 testes falharam durante a validação do classificador v4.1. Esta investigação precisa descobrir o que aconteceu em cada um.

---

## Ambiente

- **N8N DEV:** http://76.13.172.17:5678
- **Webhook:** http://76.13.172.17:5678/webhook/dev-whatsapp
- **Workflow principal:** Main - Total Assistente (`hLwhn94JSHonwHzl`)
- **Workflow Fix Conflito v2:** `ImW2P52iyCS0bGbQ`
- **User de teste:** Luiz Felipe, telefone 554391936205, user_id 2eb4065b-280c-4a50-8b54-4f9329bda0ff

### Supabase AI Messages (logs de conversa)
- **URL:** https://hkzgttizcfklxfafkzfl.supabase.co
- **Service Key:** eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imhremd0dGl6Y2ZrbHhmYWZremZsIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3MDIwODYwMSwiZXhwIjoyMDg1Nzg0NjAxfQ._DkH_9A7E1xe6WXOsWNKSWgsRcYJfxjhyTvpXFm23ok
- **Tabelas:** log_users_messages, log_total, resposta_ia

### Supabase Principal (dados do user, eventos, gastos)
- **URL:** https://ldbdtakddxznfridsarn.supabase.co
- **Service Key:** usar env var SUPABASE_PRINCIPAL_SERVICE_KEY
- **Tabelas:** spent, calendar, profiles

### Como enviar mensagem de teste
```bash
TIMESTAMP=$(date +%s)
MESSAGE="MENSAGEM_AQUI"
curl -s -X POST "http://76.13.172.17:5678/webhook/dev-whatsapp" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "metadata": {"display_phone_number": "554384983452", "phone_number_id": "744582292082931"},
    "contacts": [{"profile": {"name": "Luiz Felipe"}, "wa_id": "554391936205"}],
    "messages": [{"from": "554391936205", "id": "wamid.TEST_INV_'"$TIMESTAMP"'", "timestamp": "'"$TIMESTAMP"'", "type": "text", "text": {"body": "'"$MESSAGE"'"}}],
    "field": "messages"
  }'
```

### Como consultar respostas
```bash
curl -s "https://hkzgttizcfklxfafkzfl.supabase.co/rest/v1/log_users_messages?user_phone=eq.554391936205&order=created_at.desc&limit=5" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imhremd0dGl6Y2ZrbHhmYWZremZsIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3MDIwODYwMSwiZXhwIjoyMDg1Nzg0NjAxfQ._DkH_9A7E1xe6WXOsWNKSWgsRcYJfxjhyTvpXFm23ok" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imhremd0dGl6Y2ZrbHhmYWZremZsIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3MDIwODYwMSwiZXhwIjoyMDg1Nzg0NjAxfQ._DkH_9A7E1xe6WXOsWNKSWgsRcYJfxjhyTvpXFm23ok"
```

---

## Os 3 testes a investigar

### TESTE X1 — "Almocei hoje 45 reais"

**O que aconteceu:**
- Mensagem enviada: `Almocei hoje 45 reais`
- Resposta da IA: `"Me conta o que foi e quanto custou"`
- Timestamp: 2026-03-27T17:33:37Z
- O registro está no Supabase (log_users_messages id correspondente)

**O que deveria ter acontecido:**
- Antes dos testes, essa mensagem funcionava (5/5 PASS em testes anteriores no mesmo dia, 14:21 UTC)
- "almocei" é verbo no passado + "45 reais" é valor = deveria registrar gasto

**O que investigar:**
1. Reproduzir: enviar "Almocei hoje 45 reais" 5 vezes com 12s de intervalo. Registrar cada resposta.
2. Se falhar de novo, enviar variações pra entender o padrão:
   - "almocei 45 reais" (sem "hoje")
   - "gastei 45 no almoço hoje"
   - "almocei hoje, 45 reais"
3. Verificar se "hoje" está causando conflito — comparar taxa de sucesso com e sem "hoje"

**Contexto dos testes anteriores (mesmo dia, 14:21 UTC, ANTES da mudança de prompt):**
```
[14:21:05] "Almocei hoje 45 reais" → Gasto registrado! Almoço R$45 Alimentação ✓
[14:21:19] "Almocei hoje 45 reais" → Gasto registrado! Almoço R$45 Alimentação ✓
[14:21:30] "Almocei hoje 45 reais" → Gasto registrado! Almoço R$45 Alimentação ✓
[14:21:43] "Almocei hoje 45 reais" → Gasto registrado! Almoço R$45 Alimentação ✓
[14:21:55] "Almocei hoje 45 reais" → Gasto registrado! Almoço R$45 Alimentação ✓
```
5/5 funcionava antes. Agora falhou 1/1. Pode ser regressão do prompt ou instabilidade isolada.

---

### TESTE A2 — "Falar com Maria amanhã às 9h"

**O que aconteceu:**
- Mensagem enviada: `Falar com Maria amanhã às 9h`
- Resposta da IA: NENHUMA — nada foi registrado no Supabase
- O webhook retornou HTTP 200 (workflow started)
- Não existe nenhum registro no log_users_messages para essa mensagem em NENHUMA data

**O que deveria ter acontecido:**
- Deveria criar evento "Falar com Maria" amanhã às 09:00

**O que investigar:**
1. Reproduzir: enviar "Falar com Maria amanhã às 9h" 3 vezes. Verificar se registra no Supabase.
2. Se não registrar, o problema é no workflow N8N (não no classificador). Possibilidades:
   - Check Message Age rejeitou (timestamp do webhook pode estar errado)
   - Bot Guard (Redis) bloqueou (anti-loop)
   - Erro silencioso no AI Agent (timeout, JSON malformado)
   - O workflow Main processou mas Fix Conflito v2 não executou
3. Verificar se outros testes sem resposta (S2 "Muda a reunião pra quinta") têm o mesmo padrão
4. Se possível, verificar execuções no N8N API: `GET /api/v1/executions?workflowId=ImW2P52iyCS0bGbQ&limit=20` (precisa de API key)

**Nota:** Outros testes com "amanhã" funcionaram (GA1 "Pagar 100 reais amanhã pro Luan" = PASS, A4 "Buscar as crianças amanhã 11h30" = PASS). Então "amanhã" não é o problema. Pode ser timing, concorrência, ou algo específico da frase.

---

### TESTE R1 — "Academia segunda quarta e sexta 6h"

**O que aconteceu:**
- Mensagem enviada: `Academia segunda quarta e sexta 6h`
- Resposta da IA: NENHUMA — nada foi registrado no Supabase
- O webhook retornou HTTP 200

**O que deveria ter acontecido:**
- Deveria criar evento recorrente (WEEKLY;BYDAY=MO,WE,FR às 06:00)

**Contexto histórico:**
- A versão com "toda" funciona: em 17/03 "academia toda segunda quarta e sexta às 7h" → evento recorrente criado com sucesso
- A versão sem "toda" (recorrência implícita) é uma regra NOVA do classificador v4.1

**O que investigar:**
1. Reproduzir: enviar "Academia segunda quarta e sexta 6h" 3 vezes. Verificar se registra.
2. Se não registrar, mesmo padrão de A2 (erro silencioso no workflow)
3. Se registrar mas não como recorrente, verificar qual branch o classificador escolheu:
   - Se `criar_evento_agenda` → classificador não reconheceu recorrência implícita
   - Se `criar_evento_recorrente` → o prompt 11-recorrente.txt falhou ao processar
4. Testar com "toda": "Academia toda segunda quarta e sexta 6h" — se funcionar, confirma que o problema é a recorrência implícita
5. Testar R2 que FUNCIONOU: "Inglês terça e quinta 19h" funcionou como recorrente. Comparar: R2 tem 2 dias, R1 tem 3. Pode ser que 3 dias confunda o classificador ou o prompt.

---

## Padrão comum entre A2 e R1

Ambos retornaram "SEM RESPOSTA NO SUPABASE". Isso sugere que o problema pode não ser do classificador, mas do workflow N8N. Possibilidades:

1. **Timing:** As mensagens foram enviadas muito próximas de outras e o workflow não processou (fila, concorrência)
2. **Check Message Age:** O timestamp do webhook pode ter sido rejeitado como "mensagem antiga"
3. **Bot Guard (Redis):** Anti-loop pode ter bloqueado por mensagens repetidas no mesmo período
4. **Erro silencioso:** O AI Agent pode ter dado timeout ou retornado JSON inválido, e o workflow não registrou no Supabase

Para distinguir: se a reprodução funcionar (mensagem registra), o problema original foi de timing/concorrência. Se não funcionar, é bug persistente.

---

## Entregável

Após investigar, gere um relatório com:

| Teste | Reproduziu? | Taxa (X/N) | Causa identificada | É regressão? |
|-------|------------|------------|--------------------|--------------|

Para cada teste que falhar na reprodução, identifique em qual camada o problema está:
1. Webhook (mensagem nem chegou)
2. Check Message Age (rejeitou timestamp)
3. Bot Guard Redis (bloqueou)
4. Classificador (mandou pro branch errado)
5. AI Agent (recebeu prompt certo mas respondeu errado)
6. Log (processou mas não registrou no Supabase)
