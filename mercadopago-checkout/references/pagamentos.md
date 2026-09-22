# POST /v1/payments — campos, status e recusas

## Headers

| Header | Observação |
|---|---|
| `Authorization: Bearer <ACCESS_TOKEN>` | o prefixo (`APP_USR-`, `TEST-`) serve de diagnóstico, **não** para decidir o ambiente — conta de teste também emite credencial de produção |
| `Content-Type: application/json` | |
| `X-Idempotency-Key: <UUID>` | obrigatório para pagamentos e reembolsos desde 09/01/2024 |

Repetir a mesma chave **com o mesmo corpo** não cria segundo pagamento — é como
se sobrevive a um timeout sem cobrar duas vezes.

Repetir a chave **com corpo diferente** é erro: uma nova tentativa do comprador,
com token novo, precisa de chave nova.

Os códigos de conflito documentados (409 enquanto a primeira processa, 422 para
chave reutilizada com corpo divergente) estão descritos na documentação de
idempotência do Wallet Connect. Trate a resposta pelo código e corpo que
chegarem, sem assumir que o contrato é idêntico em `/v1/payments`.

Tempo de retenção da chave: não publicado.

## Corpo

| Campo | Tipo | Situação |
|---|---|---|
| `transaction_amount` | number | obrigatório — **em reais**, decimal |
| `payment_method_id` | string | obrigatório (`visa`, `master`, `pix`, `bolbradesco`…) |
| `token` | string | obrigatório para cartão — vem do front, uso único |
| `installments` | integer | obrigatório para cartão |
| `issuer_id` | number/string | banco emissor |
| `payer.email` | string | obrigatório |
| `payer.identification.type` / `.number` | string | obrigatórios no Brasil (`CPF`) |
| `payer.first_name` / `.last_name` | string | recomendados |
| `external_reference` | string | identificador do seu pedido — use sempre |
| `description` | string | descrição do produto |
| `statement_descriptor` | string | texto na fatura do comprador |
| `notification_url` | string | webhook; convive com o configurado no painel |
| `date_of_expiration` | string ISO 8601 | limites variam por meio |
| `additional_info.items[]` | array | `id`, `title`, `description`, `picture_url`, `category_id`, `quantity`, `unit_price` |

`additional_info` também aceita `payer` (telefone, endereço, data de cadastro,
primeira compra) e `shipments` (endereço de entrega). Nada disso é obrigatório —
são dados antifraude que melhoram a taxa de aprovação.

A tabela formal de tipos e obrigatoriedade vive só na API Reference, que é
renderizada por JavaScript e não sai em fetch simples.

## status

| status | Significado |
|---|---|
| `approved` | aprovado e creditado |
| `authorized` | autorizado, aguardando captura |
| `in_process` | em análise |
| `pending` | aguardando ação ou pagamento |
| `rejected` | rejeitado |
| `cancelled` | cancelado ou expirado |
| `refunded` | reembolsado |
| `charged_back` | contestação revertida pelo emissor (chargeback) — não confundir com `refunded`, que é devolução feita pelo lojista |
| `in_mediation` | em disputa |

## status_detail por status

Lista **não exaustiva** — o provedor acrescenta valores. Valor desconhecido é
estado não conclusivo: registre e reconcilie, nunca libere o pedido por padrão.

- `approved` → `accredited`, `partially_refunded`
- `authorized` → `pending_capture`
- `in_process` → `offline_process`, `pending_contingency`, `pending_review_manual`, `deferred_retry`
- `pending` → `pending_waiting_transfer`, `pending_waiting_payment`, `pending_challenge`
- `cancelled` → `expired` (30 dias pendente), `by_collector`, `by_payer`
- `charged_back` → `settled`, `reimbursed`, `in_process`
- `in_mediation` → `pending`
- `refunded` → `refunded`, `by_admin`

## Recusa de cartão

**Preenchimento** — dá para pedir correção ao comprador:
`cc_rejected_bad_filled_card_number`, `cc_rejected_bad_filled_date`,
`cc_rejected_bad_filled_security_code`, `cc_rejected_bad_filled_other`.

**Emissor** — o comprador precisa agir fora da sua loja:
`cc_rejected_call_for_authorize` (autorizar o valor com o banco),
`cc_rejected_card_disabled` (cartão inativo para compra online),
`cc_rejected_insufficient_amount`, `cc_rejected_invalid_installments`,
`cc_rejected_max_attempts`, `cc_rejected_duplicated_payment`.

**Antifraude** — não explique o motivo ao comprador:
`cc_rejected_blacklist`, `cc_rejected_high_risk`, `cc_rejected_other_reason`.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/response-handling/query-results
- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/how-tos/reasons-for-rejection
- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/integration-configuration/card/integrate-via-cardform
- https://www.mercadopago.com.br/developers/pt/news/2023/01/04/Idempotency-key-usage-will-be-mandatory
- https://www.mercadopago.com.br/developers/pt/docs/wallet-connect/payment-flow/idempotency/responses
- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/additional-content/industry-data/retail
