# Prompt de Teste

## O que fazer

Envie cada mensagem abaixo para o webhook do N8N DEV. Aguarde 12 segundos entre cada envio (o contexto do user é limpo automaticamente). Após enviar todas as mensagens de cada grupo, consulte o Supabase e registre a resposta da IA para cada uma.

As mensagens do Grupo 1 devem ser enviadas **5 vezes cada**. Todas as demais, **1 vez cada**.

No final, gere uma tabela com: ID, mensagem enviada, resposta da IA (na íntegra).

---

## Ambiente

**Webhook:**
```bash
TIMESTAMP=$(date +%s)
MESSAGE="MENSAGEM_AQUI"
LABEL="LABEL_AQUI"

curl -s -o /dev/null -w "%{http_code}" \
  -X POST "http://76.13.172.17:5678/webhook/dev-whatsapp" \
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

**Consulta de respostas:**
```bash
curl -s "https://hkzgttizcfklxfafkzfl.supabase.co/rest/v1/log_users_messages?user_phone=eq.554391936205&order=created_at.desc&limit=50" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imhremd0dGl6Y2ZrbHhmYWZremZsIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3MDIwODYwMSwiZXhwIjoyMDg1Nzg0NjAxfQ._DkH_9A7E1xe6WXOsWNKSWgsRcYJfxjhyTvpXFm23ok" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imhremd0dGl6Y2ZrbHhmYWZremZsIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3MDIwODYwMSwiZXhwIjoyMDg1Nzg0NjAxfQ._DkH_9A7E1xe6WXOsWNKSWgsRcYJfxjhyTvpXFm23ok"
```

---

## Mensagens

### Grupo 1 — 5 vezes cada

| ID | Mensagem |
|----|----------|
| P1 | `Enviar mensagem Gustavo Garcia as 10h30` |
| Z1 | `Café com a Ana 15 reais segunda` |

### Grupo 2 — 1 vez cada

| ID | Mensagem |
|----|----------|
| A1 | `Ligar pro João às 15h` |
| F1 | `Paguei o almoço pro Gustavo 50 reais` |
| C1 | `Preciso lembrar de ir no dentista sexta` |
| X1 | `Almocei hoje 45 reais` |

### Grupo 3 — 1 vez cada

| ID | Mensagem |
|----|----------|
| A2 | `Falar com Maria amanhã às 9h` |
| A3 | `Passar no mercado sexta às 18h` |
| A4 | `Buscar as crianças amanhã 11h30` |
| A5 | `Lembrar de pagar o boleto dia 30` |

### Grupo 4 — 1 vez cada

| ID | Mensagem |
|----|----------|
| F2 | `Dividi a conta 30 cada` |
| F3 | `Dei 20 reais pro moleque do carro` |
| F4 | `O uber deu 35 reais` |
| F5 | `Coloquei gasolina 200` |

### Grupo 5 — 1 vez cada

| ID | Mensagem |
|----|----------|
| GF1 | `Vou pagar 100 na lavação do carro` |
| GF2 | `Pago 100 pro Luan` |

### Grupo 6 — 1 vez cada

| ID | Mensagem |
|----|----------|
| GA1 | `Pagar 100 reais amanhã pro Luan` |
| GA2 | `Comprar presente de 100 sábado` |
| GA3 | `Pago 100 pro Luan amanhã` |

### Grupo 7 — 1 vez cada

| ID | Mensagem |
|----|----------|
| B1 | `Quanto gastei hoje?` |
| B2 | `O que tenho amanhã?` |
| B3 | `Meus gastos da semana` |

### Grupo 8 — 1 vez cada

| ID | Mensagem |
|----|----------|
| E1 | `Esquece o dentista` |
| E2 | `Deixa pra lá o treino de amanhã` |
| E3 | `Não preciso mais da reunião` |

### Grupo 9 — 1 vez cada

| ID | Mensagem |
|----|----------|
| R1 | `Academia segunda quarta e sexta 6h` |
| R2 | `Inglês terça e quinta 19h` |

### Grupo 10 — 1 vez cada

| ID | Mensagem |
|----|----------|
| D1 | `Enviar mensagem Gustavo Garcia as 10h30` |
| D2 | `Pagar 100 reais amanhã pro Luan` |
| D3 | `Reunião com Luan amanhã 16h` |

### Grupo 11 — 1 vez cada

| ID | Mensagem |
|----|----------|
| S1 | `oi` |
| S2 | `Muda a reunião pra quinta` |
| S3 | `Cancela o dentista de sexta` |
| S4 | `Recebi 5000 do freela` |
| S5 | `Treino todo dia 7h` |
