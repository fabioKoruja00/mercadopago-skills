# PIX e boleto

## PIX

Corpo mínimo: `transaction_amount`, `description`, `payment_method_id: "pix"`,
`payer.email`, `payer.identification.type`, `payer.identification.number`.

Na resposta:

| Campo | Conteúdo |
|---|---|
| `point_of_interaction.transaction_data.qr_code` | copia e cola |
| `point_of_interaction.transaction_data.qr_code_base64` | imagem PNG em base64, para `<img src="data:image/png;base64,…">` |
| `point_of_interaction.transaction_data.ticket_url` | página pronta do Mercado Pago com QR e copia e cola |

Expiração padrão **24 horas**. Ajuste por `date_of_expiration` (ISO 8601), com
limite entre **30 minutos e 30 dias**.

O pagamento pode cair depois do prazo do seu contador na tela, dependendo do
PSP. Nunca libere pelo relógio do front: consulte o estado na API.

## Boleto

`payment_method_id: "bolbradesco"`.

`payer` exige, além de `email`, `first_name`, `last_name` e
`identification` (`CPF` + número), o **endereço completo**:

| Campo | Exemplo |
|---|---|
| `address.zip_code` | `88000000` |
| `address.street_name` | `Rua Exemplo` |
| `address.street_number` | `123` — use `"S/N"` se não houver |
| `address.neighborhood` | `Centro` |
| `address.city` | `Curitiba` |
| `address.federal_unit` | `PR` — UF com 2 letras |

Nasce com status `pending`. A URL do boleto vem em
`transaction_details.external_resource_url`.

Expiração padrão **3 dias**, ajustável de 1 a 30. A aprovação leva até 2 horas
úteis depois do pagamento — por isso 3 dias é o mínimo recomendado. Boleto não
pago após o vencimento gera devolução automática ao comprador.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/integration-configuration/integrate-pix
- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/integration-configuration/other-payment-methods
