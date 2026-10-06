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

Na tela, o QR (`qr_code_base64`) e um botão que copia `qr_code` bastam. Mostre
o texto do copia e cola só se `navigator.clipboard` falhar, já selecionado e
anunciado em `role="status"`, para quem não consegue copiar ter alternativa.

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

São **dois prazos diferentes**, e confundi-los gera expectativa errada:

- **Vencimento** — quanto tempo o comprador tem para pagar. Padrão **3 dias**,
  ajustável de 1 a 30 por `date_of_expiration`.
- **Compensação** — quanto tempo o pagamento leva para ser reconhecido **depois**
  de pago. A documentação cita até 2 horas úteis; na prática varia com o banco.
  Não derive o vencimento desse prazo.

Boleto vencido ainda `pending` ou `in_process` deve ser cancelado quando
aplicável; a documentação informa cancelamento automático se o vencimento
ocorrer em 30 dias. Não suponha que todo boleto não pago muda de estado
imediatamente no vencimento. Se for pago após a expiração, o Mercado Pago
informa que o valor volta à conta do pagador; não libere o pedido.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/integration-configuration/integrate-pix
- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/integration-configuration/other-payment-methods
