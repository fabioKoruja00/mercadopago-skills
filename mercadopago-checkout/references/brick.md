# Payment Brick (SDK JS v2)

O Brick renderiza o formulário e **tokeniza o cartão no navegador**, de modo que
o número do cartão nunca chega ao seu servidor.

```html
<script src="https://sdk.mercadopago.com/js/v2"></script>
```

```js
const mp = new MercadoPago('<PUBLIC_KEY>', { locale: 'pt-BR' });

const brick = await mp.bricks().create('payment', 'paymentBrick_container', {
  initialization: { amount: 100 },
  customization: {
    paymentMethods: {
      creditCard: 'all',
      debitCard: 'all',
      bankTransfer: 'all',   // PIX
      ticket: 'all',         // boleto
      maxInstallments: 3,
    },
  },
  callbacks: {
    onReady: () => {},
    onSubmit: ({ selectedPaymentMethod, formData }) => { /* POST ao seu backend */ },
    onError: (erro) => console.error(erro),
  },
});
```

## Callbacks

| Callback | Papel |
|---|---|
| `onReady` | o Brick terminou de renderizar — esconder o carregando |
| `onSubmit` | **deve devolver Promise**: `resolve()` mostra sucesso, `reject()` mantém o formulário para nova tentativa |
| `onError` | todos os erros do Brick |

Ao sair da tela, chame `unmount()`. Em componente React, isso vai na limpeza do
efeito — e monte o Brick uma vez só, ou cada renderização cria outro.

## formData

Não há schema formal publicado. Na prática vem no formato do corpo de
`POST /v1/payments`: `transaction_amount`, `token`, `description`,
`installments`, `payment_method_id`, `issuer_id`, `payer.email`,
`payer.identification.type`, `payer.identification.number`.

**Repasse ao seu backend, nunca direto ao Mercado Pago.** O backend substitui o
`transaction_amount` pelo total que ele mesmo calculou, valida as parcelas e
acrescenta o `X-Idempotency-Key`. O que vem do navegador é pedido, não ordem.

Se o Brick monta uma vez e o carrinho muda depois, o `onSubmit` fecha sobre o
estado antigo. Guarde o pedido atual numa referência mutável e leia dela dentro
do callback.

## Meios de pagamento

- `creditCard` / `debitCard` / `prepaidCard`: `'all'` ou lista de ids
- `bankTransfer`: `'pix'`
- `ticket`: `'bolbradesco'`
- `mercadoPago`: `'onboarding_credits'`, `'wallet_purchase'`

Para **não** exibir um meio, remova a chave — não existe valor "desligado".

Parcelas: `minInstallments` e `maxInstallments`. O teto no Brick é conveniência
de interface: **valide as parcelas no servidor também**, senão a regra vale só
para quem usa a sua tela.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/checkout-bricks/payment-brick/default-rendering
- https://www.mercadopago.com.br/developers/pt/docs/checkout-bricks/payment-brick/payment-submission/cards
- https://www.mercadopago.com.br/developers/pt/docs/checkout-bricks/payment-brick/advanced-features/manage-payment-methods
