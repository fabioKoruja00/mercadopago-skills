# mercadopago-skills

Skills de agente para integração com o **Mercado Pago** no Brasil: Checkout
Transparente, Payment Brick, PIX, boleto, cartão e webhooks.

Escrita e testada no [Claude Code](https://claude.com/claude-code), onde a skill
é descoberta sozinha pela `description` e invocável por `/mercadopago-checkout`.

Em outras ferramentas, a descoberta automática e o comando dependem do suporte
de cada uma a skills no formato `SKILL.md` com frontmatter. Os arquivos são
Markdown comum e servem como referência mesmo onde não houver esse suporte.

## O que tem aqui

### `mercadopago-checkout`

Referência de integração. O `SKILL.md` carrega o que decide a
arquitetura; os arquivos de `references/` são lidos sob demanda, conforme a
tarefa.

| Arquivo | Conteúdo |
|---|---|
| `SKILL.md` | As regras que decidem a integração, ciclo de estado do pedido e tabela de armadilhas |
| `references/pagamentos.md` | Campos de `POST /v1/payments`, os 9 valores de `status`, exemplos de `status_detail` (lista não exaustiva), motivos de recusa de cartão |
| `references/pix-boleto.md` | Campos exigidos, caminho exato do QR Code e da URL do boleto, prazos de expiração |
| `references/webhooks.md` | Manifesto do `x-signature`, configuração no painel, o que conferir depois de validar |
| `references/brick.md` | Payment Brick (SDK JS v2): inicialização, callbacks, meios de pagamento e parcelas |
| `references/testes.md` | Cartões de teste, códigos que forçam aprovação e recusa, contas de teste |

## Instalação

Copie a pasta da skill para onde seu agente procura skills.

**Claude Code, no usuário** (vale para todos os projetos):

```bash
git clone https://github.com/fabioKoruja00/mercadopago-skills.git
cp -r mercadopago-skills/mercadopago-checkout ~/.claude/skills/
```

**Em um projeto só:**

```bash
cp -r mercadopago-skills/mercadopago-checkout <seu-projeto>/.claude/skills/
```

O agente passa a casar a skill sozinho pela `description` quando a tarefa
envolve cobrança, checkout, confirmação de pagamento ou depuração de webhook.
Para chamar à mão no Claude Code: `/mercadopago-checkout`.

## Por que existe

Os erros que derrubam uma integração com o Mercado Pago em produção quase nunca
estão em destaque na documentação. Alguns que esta skill trata de frente:

- **Valores são em reais, não em centavos** — diferente da maioria dos
  gateways. Multiplicar por 100 cobra 100× a mais, e o erro aparece na fatura
  de alguém.
- **A assinatura do webhook usa o `data.id` que vem na query**, e cada framework
  expõe esse parâmetro com um nome diferente — é a causa mais comum de `401` em
  toda notificação legítima.
- **A notificação chega repetida e fora de ordem.** Comparar só a data deixa
  passar a repetida, que vem com data igual; só a tabela de transições deixa
  passar a antiga. Precisa dos dois.
- **`X-Idempotency-Key` pertence à tentativa de pagamento**, não ao pedido.
  Repetir a chave com o mesmo corpo é o que impede a cobrança dupla no timeout;
  uma nova tentativa do comprador, com token novo, precisa de chave nova.
- **O corpo do webhook não diz o que aconteceu** em que se possa fechar um
  pedido. Depois de validar a assinatura, busque o pagamento na API e confira
  valor, moeda e `external_reference` antes de mudar qualquer estado.
- **Assinatura válida não expira sozinha.** Sem conferir o `ts` contra uma
  janela de tolerância, uma notificação capturada pode ser reenviada meses
  depois e continuará passando.

## Validade do conteúdo

Skill não se atualiza sozinha, e esta não tem um projeto a montante de onde
puxar correção: **o que envelhece aqui é a documentação do Mercado Pago**.

Cada arquivo de `references/` termina com as URLs oficiais de onde os fatos
saíram. Antes de apoiar uma decisão séria nesta skill, abra as fontes do arquivo
que você está usando e confira se ainda batem.

Os fatos foram levantados da documentação oficial em
[mercadopago.com.br/developers](https://www.mercadopago.com.br/developers/pt) em
setembro de 2026.

Nenhum valor daqui vale como credencial, e nenhuma decisão de negócio deve sair
só da leitura desta skill — teste contra o ambiente de teste do provedor.

## Licença

MIT — veja [LICENSE](LICENSE).

Este projeto não tem vínculo com o Mercado Pago. "Mercado Pago" é marca de seus
respectivos donos; aqui é citada apenas para identificar a integração descrita.
