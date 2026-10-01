# As ferramentas

Fonte: https://www.mercadopago.com.br/developers/pt/docs/mcp-server/tools
Confira a lista atual antes de invocar uma ferramenta: o servidor pode mudar.

## Leitura — seguras

### `search_documentation`
Busca um termo na documentação oficial. É a chamada de teste indicada: não toca
em nada da conta.

### `application_list`
Lista as aplicações da conta, com `AppID`, `AppName` e descrição. Serve para
descobrir o id que as outras ferramentas pedem.

### `notifications_history_diagnostics`
Diagnóstico do histórico de entrega das notificações de webhook, com métricas e falhas.

**Histórico vazio é ambíguo:** pode significar ausência de eventos ou de
configuração. Confira a configuração e o histórico na aplicação.

### `quality_checklist`
Os campos que o Mercado Pago avalia na integração. Bom para saber o que a
avaliação vai cobrar antes de rodá-la.

## Leitura que expõe segredo

### `get_credentials`
Devolve, para a aplicação escolhida:

- **Produção:** client id, client secret, access token, public key
- **Teste (sandbox):** access token e public key, de um usuário de teste
  criado automaticamente

O retorno vem em texto puro. Num agente, isso significa segredo gravado no
histórico da conversa. Redirecione para arquivo, mande ao cofre e destrua a
cópia local.

## Escrita — mudam a conta

### `create_application`
Cria aplicação. **Aplicação nova não é conta nova:** as credenciais mudam, a
conta recebedora continua a mesma. Para separar o dinheiro, é preciso outra
conta Mercado Pago.

### `save_webhook`
Grava URL e tópicos. Sobrescreve o que estiver configurado no painel, inclusive
uma integração já testada. **Salvar de novo pode gerar outra assinatura secreta**
— a antiga para de validar e toda notificação passa a dar 401.

### `create_test_user` / `add_money_test_user`
Usuário de teste e saldo. O limite documentado é de 15 contas de teste
simultâneas, e elas **não podem ser apagadas**.

## Avaliação

### `quality_evaluation`

| Campo | Tipo | Quando |
|---|---|---|
| `payment_id` | **número** | integração por Payments API (Checkout API/Pro) |
| `order_id` | **texto** | integração por Orders API |

Exige identificador de pagamento **produtivo**. A documentação oficial não
especifica aqui uma janela de sete dias; não presuma esse limite.

Informe ao menos um dos dois identificadores, com o tipo documentado.
