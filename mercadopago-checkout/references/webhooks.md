# Webhooks — assinatura e configuração

## Configurar no painel

1. **Suas integrações** → selecionar a aplicação
2. Menu esquerdo → **Webhooks** → **Configurar notificações**
3. Preencher **URL modo produção** e/ou **URL modo teste**
4. Marcar os **eventos** (para cobrança comum, só `payment`)
5. **Salvar** — é o Salvar que **gera a chave secreta**; ela não existe antes

A chave aparece na mesma tela; há botão para revelar e para redefinir.

**A chave de teste e a de produção são diferentes.** Reusar entre ambientes faz
a assinatura falhar só em produção.

O painel guarda histórico das últimas notificações com o JSON enviado, o status
da entrega e o motivo da falha. É por ali que se diagnostica.

Use a URL **estável** do ambiente, não uma que carregue hash de build — ela
morre no deploy seguinte e as notificações passam a cair no vazio.

## Assinatura x-signature

Header recebido:

```
x-signature: ts=1704908010,v1=618c85345248dd820d5fd456117c2ab2ef8eda45a0282ff693eac24131a5e839
x-request-id: <identificador da requisição>
```

Manifesto — **a ordem e os separadores fazem parte da assinatura**:

```
id:<data.id>;request-id:<x-request-id>;ts:<ts>;
```

HMAC-SHA256 do manifesto com a chave secreta, comparado em **tempo constante**
com o `v1`. Comparação com `==` vaza informação por tempo de resposta.

### O ponto que mais derruba

A notificação traz `data.id` **na query** (`?data.id=…&type=payment`) e também
no corpo, e as duas grafias podem divergir. O exemplo oficial em PHP usa
`$_GET['data_id']`; parte das implementações usa o do corpo.

Não aposte: monte o manifesto com **cada candidato** e aceite se algum bater.
Todos passam pelo mesmo HMAC com o mesmo segredo, então testar mais de um não
enfraquece a validação — só evita recusar notificação legítima.

Normalize o id para minúsculas antes de montar o manifesto.

## Depois de validar

A assinatura prova que a notificação veio do Mercado Pago. **Não prova o que
aconteceu** — o corpo traz só um identificador.

1. Busque o pagamento em `GET /v1/payments/{id}`
2. Confira `currency_id`, `transaction_amount` e `external_reference` contra o
   seu pedido
3. Só então mude o estado

## Filtrar o tipo

Chegam notificações de `merchant_order` para o mesmo pagamento, com outro
identificador. Buscar esse id em `/v1/payments` devolve 404 a cada notificação.
Filtre por `type == 'payment'` e responda 200 ao resto.

## Responder

Responda rápido, com 2xx. Processamento demorado dentro do handler leva a
timeout, e timeout vira reentrega — por isso a idempotência não é opcional.

## Em desenvolvimento

O Mercado Pago não alcança `localhost` nem IP privado, e não avisa. Use túnel
público, ou o botão de simular do painel.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/additional-content/your-integrations/notifications/webhooks
- https://www.mercadopago.com.br/developers/pt/docs/checkout-pro/payment-notifications
