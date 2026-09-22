# Testes

## Cartões

| Bandeira | Número | CVV | Validade |
|---|---|---|---|
| Mastercard | 5480 8328 0103 3311 | 123 | 11/30 |
| Visa | 4235 6477 2802 5682 | 123 | 11/30 |
| Amex | 3753 651535 56885 | 1234 | 11/30 |
| Elo débito | 5067 7667 8388 8311 | 123 | 11/30 |

## Forçar o resultado

O resultado vem do **nome do titular**, com CPF `12345678909`:

| Nome | Resultado |
|---|---|
| `APRO` | aprovado |
| `OTHE` | recusado por erro geral |
| `CONT` | pendente |
| `CALL` | recusado, precisa autorizar com o banco |
| `FUND` | recusado por saldo insuficiente |
| `SECU` | recusado por CVV inválido |
| `EXPI` | recusado por validade |
| `FORM` | recusado por erro de formulário |

Teste a recusa, não só a aprovação. O caminho de erro é o que aparece para o
comprador num dia ruim.

## Contas de teste

**Suas integrações** → aplicação → **Criar conta de teste**. Tipos: vendedor,
comprador, integrador. Vendedor e comprador devem ser do mesmo país.

Limite de **15 contas simultâneas**, e elas **não podem ser apagadas**. Cada uma
recebe usuário, senha e código de verificação de 6 dígitos.

Logado como conta de teste, você não enxerga "Credenciais de Teste" nem
"Qualidade da Integração".

## Três credenciais que parecem a mesma

Confundi-las custa horas, porque o erro não diz qual está errada.

| Credencial | Onde | Prefixo | Para quê |
|---|---|---|---|
| Produção da conta real | Aplicação > Credenciais de produção | `APP_USR-` | cobrar de verdade |
| **Teste da conta real** | Aplicação > **Detalhes da aplicação > Credenciais**, ou *Seu negócio > Configurações > Gestão e Administração > Credenciais* | `TEST-` | **testar cartão, PIX e boleto** |
| Produção da conta de **teste** | dados do usuário de teste criado no painel | `APP_USR-` | fluxos de redirecionamento e carteira |

A terceira engana: vem com o mesmo prefixo da primeira e é oferecida junto das
contas de teste, o que faz parecer a credencial de teste. **Não é.** Usá-la em
`POST /v1/payments` devolve:

```json
{"message":"Unauthorized use of live credentials","error":"unauthorized","status":401,"cause":[{"code":7}]}
```

A mensagem sugere problema de permissão ou de ambiente, e não há nada errado com
o token — ele autentica normalmente em `GET /users/me`. O que falta é ser a
credencial certa para o tipo de teste.

A documentação é direta: para testar cartão é preciso **trocar as credenciais de
produção da conta de teste vendedora pelas credenciais de teste da conta real**.

## Checkout Bricks não aceita conta de teste

A documentação avisa em destaque: **integrações com Checkout Bricks não suportam
contas de teste**. Criar vendedor e comprador de teste e cobrar com o token
deles devolve `401` com `code: 7`, "Unauthorized use of live credentials" — a
mensagem não diz a causa real, e faz procurar erro de credencial onde não há.

Com Bricks, o teste é feito com as **credenciais de teste da aplicação**
(prefixo `TEST-`, na aba *Credenciais de teste* do painel) mais os cartões de
teste desta página.

Cuidado para não confundir com o que algumas ferramentas chamam de "sandbox": o
token de um usuário de teste criado automaticamente vem com prefixo `APP_USR-`
e **não** substitui a credencial de teste da aplicação.

## Qual credencial usar

| Cenário | Credencial |
|---|---|
| Cartão, PIX e boleto pelo Brick | **credenciais de teste** da sua conta real |
| Fluxo com redirecionamento ou carteira | duas contas de teste (vendedor e comprador), com as credenciais de produção da conta de teste vendedora |

Não use o e-mail do usuário de teste no campo de e-mail do Brick.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/your-integrations/test/cards
- https://www.mercadopago.com.br/developers/pt/docs/your-integrations/test/accounts
- https://www.mercadopago.com.br/developers/pt/docs/checkout-bricks/integration-test/test-payment-flow
