---
name: mercadopago-payment-brick
description: >-
  Montar, revisar e depurar a interface Payment Brick do Mercado Pago:
  SDK JS, Public Key, onSubmit, meios de pagamento, parcelas e ciclo de
  vida do componente. Use em tarefas de frontend do Checkout Bricks.
---

# Mercado Pago — Payment Brick

Fonte oficial: https://www.mercadopago.com.br/developers/pt/docs/checkout-bricks/payment-brick/default-rendering

O Brick coleta os dados de pagamento e, quando o meio é cartão, tokeniza
os dados do cartão no navegador. A
Public Key é pública; o Access Token fica no backend. Antes de criar o Brick,
o servidor deve ter um pedido válido, com valor e meios permitidos definidos.

Leia [a referência de integração](references/brick.md) para inicialização,
callbacks, opções de pagamento e desmontagem do componente. Em `onSubmit`,
devolva a Promise do pedido ao backend. O backend reconstrói valor, referência
e itens a partir do pedido e valida meio e parcelas; não encaminhe o
`formData` inteiro diretamente à API. O backend controla a idempotência
das tentativas, inclusive após timeout ou reenvio de `onSubmit`.

`pending` e `in_process` pedem telas e acompanhamento assíncrono por
consulta ou webhook. Resolver a Promise de `onSubmit` apenas conclui o fluxo
do componente; não confirma o pagamento nem libera o pedido. Para criar e consultar
pagamentos, use `mercadopago-checkout`; para notificações, use
`mercadopago-webhooks`.
