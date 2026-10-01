---
name: mercadopago-checkout
description: >-
  Criar ou revisar pagamentos pelo Checkout API via Payments API do Mercado
  Pago no Brasil: PIX, boleto, cartão, valor, idempotência, tentativas e
  estados. Use para cobrança, pagamento duplicado, valor errado ou recusa.
  Para interface Payment Brick e recepção de webhook, use as skills específicas.
---

# Mercado Pago — Payments API

Base oficial: https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/
Leia a referência do meio de pagamento em uso; requisitos e prazos variam.

## Regras que decidem o pagamento

1. O servidor calcula o valor a partir do pedido e catálogo; não confie em
   `transaction_amount` enviado pelo navegador. Na Payments API, valores
   monetários são expressos em reais (ex.: `10.50` = R$ 10,50).
2. Persista pedido e tentativa antes de chamar `POST /v1/payments`. Envie o
   identificador do pedido em `external_reference` para a reconciliação.
3. `X-Idempotency-Key` pertence à tentativa. Se a mesma chamada deu timeout,
   repita a **mesma chave e o mesmo corpo**. Se o comprador fizer nova tentativa
   após recusa, gere novo token de cartão e nova chave. Não use uma chave nova
   para repetir uma chamada cujo resultado ainda é desconhecido.
4. `pending` e `in_process` não comprovam recebimento. Registre o
   `status` devolvido e acompanhe-o pela API; liberações dependem do estado
   conferido e das regras do pedido.
5. A public key pode ir ao navegador; access token fica no servidor. Com
   frameworks que embutem ambiente no build, confirme que o token usado pela
   função seja lido em tempo de execução.

## Diagnóstico

| Sintoma | Confira |
|---|---|
| Valor 100× errado | `transaction_amount` em reais, sem conversão para centavos |
| Cobrança duplicada após timeout | mesma chave e corpo no retry técnico |
| Chave recusada | nova tentativa do comprador precisa de chave nova |
| Cartão recusado no retry | token de cartão deve ser gerado de novo |
| Pedido preso em pendente | consulte o pagamento, não o relógio da interface |
| Status desconhecido | registre e reconcilie; não libere por padrão |
| Estorno parcial ou contestação | trate `status_detail`, `refunded` e `charged_back` conforme o pagamento consultado |

Para o Payment Brick, leia `mercadopago-payment-brick`. Para assinatura,
notificação e reconciliação de webhooks, leia `mercadopago-webhooks`.

## Referências

- [Pagamentos, status e recusas](references/pagamentos.md)
- [PIX e boleto](references/pix-boleto.md)
- [Cartões e credenciais de teste](references/testes.md)
