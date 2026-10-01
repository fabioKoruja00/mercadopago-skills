# mercadopago-skills

Skills independentes para integrar o Mercado Pago no Brasil. São arquivos
`SKILL.md` com referências lidas conforme a tarefa, compatíveis com agentes
que adotam esse formato.

| Skill | Use para |
|---|---|
| [mercadopago-checkout](mercadopago-checkout/SKILL.md) | Payments API: PIX, boleto, cartão, valor, tentativas e estados |
| [mercadopago-payment-brick](mercadopago-payment-brick/SKILL.md) | interface Payment Brick, callbacks, meios e parcelas |
| [mercadopago-webhooks](mercadopago-webhooks/SKILL.md) | assinatura, notificações, reentrega e conciliação |
| [mercadopago-mcp-server](mercadopago-mcp-server/SKILL.md) | instalação e uso do MCP Server oficial |

As referências técnicas ficam em `references/` dentro de cada skill. Cada
arquivo cita as páginas usadas. A divisão evita carregar instruções de
frontend ou webhooks quando a tarefa é apenas criar uma cobrança.

## Instalação

Copie as quatro pastas `mercadopago-*` para o diretório global de skills do
agente, por exemplo `~/.codex/skills/` ou `~/.claude/skills/`. Para instalar
apenas em um projeto, copie para o diretório de skills desse projeto. A
descoberta automática depende do suporte do agente a `SKILL.md`.

O MCP Server do Mercado Pago é um serviço separado das skills; instalar os
arquivos não conecta nem autoriza o MCP.

## Fontes e limites

O conteúdo foi redigido a partir de documentação pública do
[Mercado Pago Developers](https://www.mercadopago.com.br/developers/pt),
levantada em setembro de 2026 e revisada em outubro de 2026. Fontes principais:

- [Checkout API via Payments API](https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/)
- [Payment Brick](https://www.mercadopago.com.br/developers/pt/docs/checkout-bricks/payment-brick/default-rendering)
- [Webhooks](https://www.mercadopago.com.br/developers/pt/docs/checkout-api-payments/additional-content/your-integrations/notifications/webhooks)
- [Ferramentas do MCP Server](https://www.mercadopago.com.br/developers/pt/docs/mcp-server/tools)
- [Cartões de teste](https://www.mercadopago.com.br/developers/pt/docs/your-integrations/test/cards)

Estas skills resumem decisões de integração e não reproduzem nem substituem
a documentação oficial. Produtos, parâmetros, prazos e ferramentas podem
mudar; confira a página vinculada antes de implementar ou publicar pagamentos.
Nenhuma credencial deve ser incluída no repositório nem em conversas.

## Licença e autoria

[MIT](LICENSE). **Projeto independente e não oficial.** Estas skills não
são produzidas, mantidas, aprovadas nem endossadas pelo Mercado Pago e não
têm vínculo com a empresa. A marca é citada apenas para identificar a
integração descrita.
