---
name: mercadopago-mcp-server
description: >-
  Instalar, autenticar e usar o MCP Server oficial do Mercado Pago em Claude
  Code, Claude Desktop, Cursor, VS Code, Windsurf ou Cline. Use ao configurar o
  MCP, quando o conector não aparece na lista depois de instalado, quando o
  OAuth não completa, ou ao usar as ferramentas de aplicações, credenciais,
  webhooks, usuários de teste e avaliação de qualidade da integração.
---

# MCP Server do Mercado Pago

Endpoint oficial: `https://mcp.mercadopago.com/mcp`, HTTP com OAuth.

Requisitos medidos na documentação: Node 20+, npm 5.2+ (com npx).

## Instalar

O transporte decide se vai funcionar. Prefira **stdio com `mcp-remote`**:

```bash
claude mcp add --scope user mercadopago -- npx -y mcp-remote@latest https://mcp.mercadopago.com/mcp
```

No **Windows**, o `.cmd` não executa como processo em vários clientes — envolva
em `cmd /c`:

```bash
claude mcp add --scope user mercadopago -- cmd /c npx -y mcp-remote@latest https://mcp.mercadopago.com/mcp
```

O registro HTTP direto (`--transport http`) é aceito e aparece na lista, mas
exige um OAuth interativo que nem todo cliente consegue completar — veja
[references/instalacao.md](references/instalacao.md).

## Autenticar

`mcp-remote` abre o navegador, recebe o retorno em `127.0.0.1` e **grava o token
em `~/.mcp-auth`**. Depois disso qualquer cliente que use a mesma ponte conecta
sem repetir o login.

Rodar a ponte à mão é o diagnóstico mais rápido, porque imprime a URL e o erro:

```bash
npx -y mcp-remote@latest https://mcp.mercadopago.com/mcp
```

Saída de sucesso: `Connected to remote server` e `Proxy established successfully`.

**Isso dispara o fluxo de autorização de verdade** — abre o navegador e concede
acesso à conta. O escopo pedido é `offline_access write read`: o MCP passa a
**escrever** na conta, podendo criar aplicação e alterar webhook. Não rode como
"só um teste" sem querer conceder isso.

## As 10 ferramentas documentadas

Confira a [lista oficial atual](https://www.mercadopago.com.br/developers/pt/docs/mcp-server/tools) antes de usar: nomes e disponibilidade podem mudar.

| Ferramenta | O que faz |
|---|---|
| `search-documentation` | busca na documentação oficial — leitura pura; confirme o nome exposto pelo cliente |
| `application_list` | lista as aplicações da conta |
| `create_application` | cria aplicação |
| `get_credentials` | client id/secret, access token e public key, produção **e** teste |
| `save_webhook` | grava a URL e os tópicos do webhook |
| `notifications_history_diagnostics` | diagnóstico do histórico de notificações |
| `quality_checklist` | os campos que o Mercado Pago avalia |
| `quality_evaluation` | avalia a integração a partir de um pagamento |
| `create_test_user` | cria usuário de teste |
| `add_money_test_user` | põe saldo no usuário de teste |

Detalhe de cada uma, com o que medi dos schemas, em
[references/ferramentas.md](references/ferramentas.md).

## Cuidados

**`get_credentials` devolve segredo em texto.** Num agente, isso entra no
histórico da conversa e fica gravado. Redirecione para arquivo, mande ao cofre e
destrua a cópia — nunca deixe o valor aparecer na resposta.

**`create_application`, `save_webhook` e `create_test_user` escrevem na conta
real.** Uma aplicação já configurada e testada pode ser alterada por uma chamada
distraída. Em conta com integração em produção, prefira as ferramentas de
leitura e confirme antes de escrever.

**`quality_evaluation` usa pagamento produtivo real**, conforme a documentação
oficial. Use `payment_id` (número) para Payments API e `order_id` (texto)
para Orders API. Não envie um pagamento de teste nem afirme prazo máximo sem
consultar a documentação vigente.

## Quando o conector não aparece

São listas diferentes, e instalar numa não põe na outra:

| Cliente | Arquivo |
|---|---|
| Claude Code | `~/.claude.json` |
| Claude Desktop | `claude_desktop_config.json` |

O cliente carrega os MCPs **ao iniciar**: adicionar com a sessão aberta não
registra nada. Reinicie o processo inteiro, não só a janela.

Roteiro de diagnóstico em [references/diagnostico.md](references/diagnostico.md).
