---
name: mercadopago-checkout
description: >-
  Integração com Mercado Pago no Brasil — Checkout Transparente (Checkout API),
  Payment Brick, PIX, boleto e cartão, webhooks com assinatura x-signature,
  idempotência e ciclo de estado do pedido. Use ao criar ou revisar cobrança,
  checkout, confirmação de pagamento ou conciliação com Mercado Pago em qualquer
  linguagem. Também ao depurar webhook recusado, pagamento duplicado, valor
  errado ou pedido que não fecha.
---

# Mercado Pago — Checkout Transparente e webhooks

Referência de integração no Brasil. Os fatos vêm da documentação oficial
(`mercadopago.com.br/developers`); cada arquivo em `references/` cita a fonte.

## Quando usar

- Criar cobrança por PIX, boleto ou cartão.
- Montar ou revisar checkout com Payment Brick.
- Receber e validar webhook de pagamento.
- Depurar: webhook com 401, pagamento duplicado, valor 100× errado, pedido
  preso em pendente, recusa de cartão sem motivo claro.

## As cinco regras que decidem a integração

**1. O navegador nunca define o preço.** Ele manda identificador e quantidade;
o servidor remonta o pedido a partir do catálogo e calcula o total. Aceitar
`transaction_amount` do cliente é aceitar o preço que ele digitar.

**2. Valores são em REAIS, não em centavos.** `10.50` é dez reais e cinquenta.
Diferente de Stripe e da maioria dos gateways — multiplicar por 100 cobra 100×
a mais, e o erro só aparece na fatura de alguém.

**3. Grave o pedido ANTES de chamar a API.** Pedido órfão, que nunca virou
pagamento, é descartável. Pagamento órfão, sem pedido, é dinheiro cobrado sem
registro de quem comprou o quê. Mande o identificador do pedido em
`external_reference`: é o que liga os dois depois.

**4. `X-Idempotency-Key` é obrigatório e deve ser guardado com o pedido.** Gerar
um UUID novo a cada tentativa transforma retry em cobrança dupla. A chave vive
no pedido, não na requisição.

**5. O webhook é a fonte de verdade, e não se confia no corpo dele.** Valide a
assinatura, depois **busque o pagamento na API** e confira valor, moeda e
`external_reference` antes de mudar qualquer estado. O corpo da notificação
carrega só o identificador; tratá-lo como o estado é aceitar o que um
desconhecido mandou.

## Estado do pedido

Mapeie o `status` do provedor para estados seus e declare as transições
permitidas. Não deixe o provedor ditar o vocabulário do seu domínio.

A notificação **chega repetida e fora de ordem**. Para resistir aos dois:

- guarde a data de atualização que veio do provedor (`date_last_updated`);
- só aplique se a data recebida for **mais recente** que a guardada;
- **e** se a transição existir na sua tabela.

Só a data deixa passar a notificação repetida, que chega com data igual. Só a
tabela deixa passar a antiga. Precisa dos dois. `pago → estornado` é válido;
`pago → aguardando` não.

## Armadilhas que derrubam em produção

| Sintoma | Causa | Conserto |
|---|---|---|
| Webhook 401 em toda notificação | manifesto montado com o `data.id` errado — query e corpo divergem em grafia | conferir os dois candidatos contra o mesmo HMAC; tentar mais de um não afrouxa nada |
| Assinatura só falha em produção | segredo de teste e de produção são **diferentes** por aplicação | pegar o segredo do ambiente ativo, não reusar |
| Cobrança dupla no retry | `X-Idempotency-Key` nova a cada tentativa | chave presa ao pedido |
| Valor 100× errado | tratado como centavos | reais, decimal |
| Pedido não fecha com pagamento aprovado | só `status` foi olhado | `approved` pode vir com `status_detail` que exige ação; ver `references/pagamentos.md` |
| Webhook nunca chega em dev | o Mercado Pago não alcança localhost | túnel público, ou o botão de simular no painel |
| 404 a cada notificação | `merchant_order` tratado como pagamento | filtrar `type == 'payment'` |
| Token de cartão recusado no retry | token do front é de uso único | token novo a cada tentativa |
| Boleto "demora demais" | compensação bancária leva até 3 dias úteis | não é bug; avisar o comprador |
| Estorno não reflete | `refunded`/`charged_back` tratados como "não aprovado" | são estados próprios, com efeito próprio |

## Segurança

- Access token e segredo de webhook são **server-side**. Só a public key vai ao
  navegador.
- Em framework que substitui variáveis no build (Astro, Vite, Next), garanta
  leitura em **tempo de execução** — senão a função sobe sem token e falha só
  em produção.
- Sem segredo de webhook configurado, **recuse** a notificação. Fail-open aqui
  é aceitar qualquer um dizendo que um pagamento foi aprovado.

## Referências

| Arquivo | Conteúdo |
|---|---|
| [references/pagamentos.md](references/pagamentos.md) | Campos de `POST /v1/payments`, `status`, `status_detail`, motivos de recusa |
| [references/pix-boleto.md](references/pix-boleto.md) | PIX e boleto: campos exigidos, onde vem QR Code e URL, expiração |
| [references/webhooks.md](references/webhooks.md) | Assinatura `x-signature`, manifesto, configuração no painel, idempotência |
| [references/brick.md](references/brick.md) | Payment Brick: inicialização, callbacks, meios e parcelas |
| [references/testes.md](references/testes.md) | Cartões de teste, forçar aprovação e recusa, contas de teste |
