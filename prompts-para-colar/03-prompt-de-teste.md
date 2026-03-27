# Prompt de Teste — Bateria Completa para Validar Classificador v4.1

Cole este prompt inteiro em outra IA (Claude, etc.) para ela executar os testes no N8N DEV.

---

## Contexto

Você vai testar o classificador de intenções do Total Assistente enviando mensagens via webhook e verificando as respostas no Supabase.

## Ambiente

- **Webhook DEV:** `http://76.13.172.17:5678/webhook/dev-whatsapp`
- **Supabase AI Messages:** `https://hkzgttizcfklxfafkzfl.supabase.co`
- **Supabase Service Key:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imhremd0dGl6Y2ZrbHhmYWZremZsIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3MDIwODYwMSwiZXhwIjoyMDg1Nzg0NjAxfQ._DkH_9A7E1xe6WXOsWNKSWgsRcYJfxjhyTvpXFm23ok`
- **User de teste:** Luiz Felipe, telefone 554391936205, email animadoluiz@gmail.com
- **Tabela de respostas:** `log_users_messages` (campo `user_message` e `ai_message`)

## Como enviar uma mensagem

```bash
TIMESTAMP=$(date +%s)
WEBHOOK_URL="http://76.13.172.17:5678/webhook/dev-whatsapp"
MESSAGE="MENSAGEM_AQUI"
LABEL="LABEL_AQUI"

curl -s -o /dev/null -w "%{http_code}" \
  -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "metadata": {
      "display_phone_number": "554384983452",
      "phone_number_id": "744582292082931"
    },
    "contacts": [
      {
        "profile": { "name": "Luiz Felipe" },
        "wa_id": "554391936205"
      }
    ],
    "messages": [
      {
        "from": "554391936205",
        "id": "wamid.TEST_'"$LABEL"'_'"$TIMESTAMP"'",
        "timestamp": "'"$TIMESTAMP"'",
        "type": "text",
        "text": { "body": "'"$MESSAGE"'" }
      }
    ],
    "field": "messages"
  }'
```

## Como consultar as respostas

```bash
SUPABASE_URL="https://hkzgttizcfklxfafkzfl.supabase.co"
SUPABASE_KEY="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imhremd0dGl6Y2ZrbHhmYWZremZsIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3MDIwODYwMSwiZXhwIjoyMDg1Nzg0NjAxfQ._DkH_9A7E1xe6WXOsWNKSWgsRcYJfxjhyTvpXFm23ok"

curl -s "$SUPABASE_URL/rest/v1/log_users_messages?user_phone=eq.554391936205&order=created_at.desc&limit=N" \
  -H "apikey: $SUPABASE_KEY" \
  -H "Authorization: Bearer $SUPABASE_KEY" | python3 -m json.tool
```

## Regras de execução

1. **Intervalo:** Esperar 12 segundos entre cada envio (o contexto do user é limpo automaticamente a cada ~10s)
2. **Verificação:** Após enviar TODOS os testes de um grupo, esperar 15s e consultar o Supabase
3. **NUNCA testar com payload de áudio** — causa problemas no workflow
4. **Output:** Para cada teste, registrar a mensagem enviada, a resposta da IA, e se o resultado está correto

## Testes a executar

### GRUPO 1 — Testes que FALHAVAM antes (5 repetições cada)
Estes são os testes que motivaram a mudança. Cada um deve ser enviado 5 vezes para medir consistência.

| ID | Mensagem | Resultado esperado | Repetições |
|----|----------|--------------------|------------|
| P1 | `Enviar mensagem Gustavo Garcia as 10h30` | Evento agendado (agenda) | 5x |
| Z1 | `Café com a Ana 15 reais segunda` | Evento agendado (agenda) com valor no nome | 5x |

### GRUPO 2 — Testes que FUNCIONAVAM antes (1x cada, smoke test)
Verificar que nada quebrou.

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| A1 | `Ligar pro João às 15h` | Evento agendado |
| F1 | `Paguei o almoço pro Gustavo 50 reais` | Gasto registrado R$50 |
| C1 | `Preciso lembrar de ir no dentista sexta` | Evento agendado |
| X1 | `Almocei hoje 45 reais` | Gasto registrado R$45 |

### GRUPO 3 — Novos cenários de AGENDA com verbos ambíguos (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| A2 | `Falar com Maria amanhã às 9h` | Evento agendado |
| A3 | `Passar no mercado sexta às 18h` | Evento agendado |
| A4 | `Buscar as crianças amanhã 11h30` | Evento agendado (NÃO busca) |
| A5 | `Lembrar de pagar o boleto dia 30` | Evento agendado |

### GRUPO 4 — Novos cenários de GASTO com contexto social (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| F2 | `Dividi a conta 30 cada` | Gasto registrado R$30 |
| F3 | `Dei 20 reais pro moleque do carro` | Gasto registrado R$20 |
| F4 | `O uber deu 35 reais` | Gasto registrado R$35 |
| F5 | `Coloquei gasolina 200` | Gasto registrado R$200 |

### GRUPO 5 — Gasto FUTURO sem data (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| GF1 | `Vou pagar 100 na lavação do carro` | Gasto registrado R$100 |
| GF2 | `Pago 100 pro Luan` | Gasto registrado R$100 |

### GRUPO 6 — Gasto FUTURO com data = agenda (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| GA1 | `Pagar 100 reais amanhã pro Luan` | Evento agendado com R$100 no nome |
| GA2 | `Comprar presente de 100 sábado` | Evento agendado com R$100 no nome |
| GA3 | `Pago 100 pro Luan amanhã` | Evento agendado com R$100 no nome |

### GRUPO 7 — Busca com data (NÃO deve virar agenda) (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| B1 | `Quanto gastei hoje?` | Listagem de gastos (busca financeira) |
| B2 | `O que tenho amanhã?` | Listagem de eventos (busca agenda) |
| B3 | `Meus gastos da semana` | Listagem de gastos (busca financeira) |

### GRUPO 8 — Exclusão com negações novas (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| E1 | `Esquece o dentista` | Exclusão/busca de evento para excluir |
| E2 | `Deixa pra lá o treino de amanhã` | Exclusão de evento |
| E3 | `Não preciso mais da reunião` | Exclusão de evento |

### GRUPO 9 — Recorrência implícita (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| R1 | `Academia segunda quarta e sexta 6h` | Evento recorrente (WEEKLY;BYDAY=MO,WE,FR) |
| R2 | `Inglês terça e quinta 19h` | Evento recorrente (WEEKLY;BYDAY=TU,TH) |

### GRUPO 10 — Dica educativa (1x cada, verificar se a dica aparece)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| D1 | `Enviar mensagem Gustavo Garcia as 10h30` | Evento agendado + dica de formato |
| D2 | `Pagar 100 reais amanhã pro Luan` | Evento agendado + dica de "depois que pagar, manda: paguei 100 pro Luan" |
| D3 | `Reunião com Luan amanhã 16h` | Evento agendado SEM dica (mensagem clara) |

### GRUPO 11 — Smoke tests de features não alteradas (1x cada)

| ID | Mensagem | Resultado esperado |
|----|----------|--------------------|
| S1 | `oi` | Saudação (padrão) |
| S2 | `Muda a reunião pra quinta` | Edição de evento |
| S3 | `Cancela o dentista de sexta` | Exclusão de evento |
| S4 | `Recebi 5000 do freela` | Gasto registrado como entrada/receita R$5000 |
| S5 | `Treino todo dia 7h` | Evento recorrente diário |

## Total: 38 testes

- Grupo 1: 10 envios (2 mensagens x 5 repetições)
- Grupos 2-11: 28 envios (1x cada)
- **Total: 38 envios**
- **Tempo estimado:** ~8 minutos (38 x 12s)

## Formato do relatório final

Gere uma tabela com:

```markdown
| # | ID | Mensagem | Resposta da IA | Esperado | Veredicto |
|---|-----|----------|---------------|----------|-----------|
```

E no final, um resumo:

```markdown
## Resumo
- Total: X/38 PASS
- Taxa: X%
- Falhas: listar IDs que falharam
- Regressões: listar testes do Grupo 2/11 que falharam (eram estáveis antes)
```
