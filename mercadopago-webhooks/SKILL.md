---
name: mercadopago-webhooks
description: >-
  Implementar e revisar notificações Webhook do Mercado Pago: assinatura
  x-signature, data.id da query, consulta do pagamento, idempotência,
  concorrência, reentrega e reconciliação. Use para webhook 401, 404,
  pagamento não confirmado ou pedido com estado incorreto.
---

# Mercado Pago — Webhooks de pagamento

Fonte oficial: https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/additional-content/your-integrations/notifications/webhooks

Leia [a referência detalhada](references/webhooks.md) antes de implementar
assinatura, resposta HTTP ou reentrega. Valide o `x-signature` com o segredo
de webhook da aplicação e do ambiente corretos, o `data.id` da query e o
`x-request-id` do header; rejeite notificações sem segredo.
Filtre o tópico esperado. A notificação é um gatilho: consulte o recurso na
API com o access token server-side e confira `external_reference`, valor,
moeda e recebedor antes de mudar o pedido. Para Payments API, consulte
`GET /v1/payments/{data.id}`; o `id` na raiz do evento identifica a
notificação. A criação do pagamento deve incluir `external_reference`.

Notificações podem chegar repetidas ou fora de ordem; eventos diferentes podem
se referir ao mesmo pagamento. A leitura do pedido,
a validação de transição e a gravação precisam ser atômicas. Se o estado
consultado for desconhecido, não libere o pedido. Confirme entrega com 2xx
somente após persistência durável; mantenha reconciliação para notificações
perdidas. O fluxo de criação da cobrança pertence à skill
`mercadopago-checkout`.
