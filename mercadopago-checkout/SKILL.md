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

**4. `X-Idempotency-Key` pertence à TENTATIVA, não ao pedido nem à requisição.**
Distinga dois casos que parecem iguais:

- **Retry técnico** — o POST deu timeout ou erro de transporte e você não sabe
  se a cobrança aconteceu. Repita **a mesma chave com o mesmo corpo**. É o que
  impede a cobrança dupla.
- **Nova tentativa do comprador** — o pagamento foi recusado e ele tenta de
  novo. Corpo diferente, token novo, **chave nova**. Reusar a chave aqui devolve
  erro, porque o corpo diverge do da primeira chamada.

Modele tentativas de pagamento ligadas ao pedido; a chave e o corpo enviado
ficam gravados na tentativa, antes do POST. Um pedido pode ter várias.

**5. O webhook é GATILHO; a fonte de verdade é o `GET /v1/payments/{id}`.**
Valide a assinatura, busque o pagamento na API e confira valor, moeda e
`external_reference` antes de mudar qualquer estado. O corpo traz metadados
(tipo, ação, data, modo), mas nenhum estado em que se possa fechar um pedido.

## Estado do pedido

Mapeie o `status` do provedor para estados seus e declare as transições
permitidas. Não deixe o provedor ditar o vocabulário do seu domínio.

A notificação **chega repetida e fora de ordem**. Para resistir aos dois:

- guarde a data de atualização que veio do provedor (`date_last_updated`);
- só aplique se a data recebida for **mais recente** que a guardada;
- **e** se a transição existir na sua tabela.

Comparação estrita (`recebida <= guardada` não aplica) já descarta a repetida e
a atrasada. A tabela existe para outro risco: duas notificações do mesmo
pagamento processadas **ao mesmo tempo**, cada uma lendo o estado antes da
outra gravar. `pago → estornado` é válido; `pago → aguardando` não.

A leitura do estado, a decisão e a gravação têm de acontecer numa **transação**
(ou com trava por pedido). Fora dela, a tabela não protege: as duas passam.

## Armadilhas que derrubam em produção

| Sintoma | Causa | Conserto |
|---|---|---|
| Webhook 401 em toda notificação | manifesto montado com o `data.id` errado | o valor vem da **query**; o que muda entre frameworks é o nome da chave, não a fonte |
| Assinatura só falha em produção | segredo de teste e de produção são **diferentes** por aplicação | pegar o segredo do ambiente ativo, não reusar |
| Cobrança dupla após timeout | chave nova no retry técnico | repetir a **mesma chave com o mesmo corpo** da chamada que deu timeout |
| Valor 100× errado | tratado como centavos | reais, decimal |
| Reembolso parcial some do pedido | `approved` tratado como valor cheio | `approved/accredited` libera; `approved/partially_refunded` exige atualizar o valor devolvido |
| Estado novo do provedor libera pedido | `switch` com `default` otimista | valor desconhecido é estado **não conclusivo**: registre e reconcilie, nunca libere |
| Webhook nunca chega em dev | o Mercado Pago não alcança localhost | túnel público, ou o botão de simular no painel |
| 404 a cada notificação | `merchant_order` tratado como pagamento | filtrar `type == 'payment'` |
| Chave de idempotência recusada | mesma chave reusada com corpo diferente | nova tentativa do comprador é **outra** tentativa: token novo, chave nova |
| Token de cartão recusado no retry | token do front é de uso único | token novo a cada nova tentativa |
| Boleto "demora demais" | confirmação não é instantânea após o pagamento | não é bug; ver o prazo em `references/pix-boleto.md` |
| Assinatura válida reaproveitada depois | `ts` do header não conferido | rejeitar fora de uma janela de tolerância; HMAC válido sem prazo vale para sempre |
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
