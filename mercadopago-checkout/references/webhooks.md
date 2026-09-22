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

O `data.id` do manifesto é o que chega **na query** (`?data.id=…&type=payment`).
O exemplo oficial em PHP lê de `$_GET`.

O que varia é **o nome da chave que o seu framework expõe** — `data.id`,
`data_id`, aninhado sob `data` —, não a fonte. Resolva lendo a query direito.

Normalize o id para minúsculas antes de montar o manifesto.

Aceitar também o id do corpo como candidato não abre brecha (todo candidato
passa pelo mesmo HMAC, e sem o segredo nenhum bate), mas é tolerar uma
divergência que não deveria existir. Prefira acertar a leitura da query.

## Replay

Assinatura válida não expira sozinha: quem capturar uma notificação legítima
pode reenviá-la meses depois, e o HMAC continua batendo.

Confira o `ts` do header contra uma **janela de tolerância** — minutos, não
horas — e recuse o que estiver fora. Some a isso a deduplicação por
identificador já processado.

## Depois de validar

A assinatura prova que a notificação veio do Mercado Pago. **Não prova o que
aconteceu** — o corpo traz metadados (tipo, ação, data, modo), mas nenhum estado
em que se possa fechar um pedido.

1. Busque o pagamento em `GET /v1/payments/{id}`
2. Confira `currency_id`, `transaction_amount` e `external_reference` contra o
   seu pedido. Com mais de uma conta ou aplicação, confira também `live_mode` e
   o recebedor esperado — e não deixe um pedido já ligado a um pagamento ser
   reassociado a outro
3. Só então mude o estado

## Concorrência

Duas notificações do mesmo pagamento podem ser processadas ao mesmo tempo, cada
uma lendo o estado antes de a outra gravar. Leitura, decisão e gravação vão numa
**transação** com trava por pedido. Sem isso, a tabela de transições não
protege: as duas passam pela mesma verificação e as duas aplicam.

## Filtrar o tipo

Chegam notificações de `merchant_order` para o mesmo pagamento, com outro
identificador. Buscar esse id em `/v1/payments` devolve 404 a cada notificação.
Filtre por `type == 'payment'` e responda 200 ao resto — responder erro faria o
provedor reentregar para sempre algo que você decidiu ignorar.

## Responder

A ordem importa mais que a pressa: **valide, persista de forma durável, e só
então responda 2xx**. Responder 2xx antes de gravar e cair em seguida perde a
notificação para sempre — o provedor considera entregue e não reenvia.

Se não deu para persistir, responda erro **de propósito**, para provocar a
reentrega.

Processamento demorado dentro do handler leva a timeout, e timeout vira
reentrega — por isso a idempotência não é opcional. Trabalho pesado (e-mail,
estoque, entrega) sai do handler para uma fila.

Mantenha um job de reconciliação para pagamentos que ficaram sem atualização:
webhook perdido não avisa que se perdeu.

## Em desenvolvimento

O Mercado Pago não alcança `localhost` nem IP privado, e não avisa. Use túnel
público, ou o botão de simular do painel.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/additional-content/your-integrations/notifications/webhooks
- https://www.mercadopago.com.br/developers/pt/docs/checkout-pro/payment-notifications
