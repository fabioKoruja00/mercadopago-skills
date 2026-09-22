# Diagnóstico

## O conector não aparece na lista

Percorra nesta ordem — cada passo elimina uma causa.

**1. Você olhou a lista certa?** Claude Code e Claude Desktop têm listas
separadas. O `/mcp` do Claude Code não mostra o que está no
`claude_desktop_config.json`, e a tela de Conectores do aplicativo não mostra o
que está no `~/.claude.json`. Instalar num lugar não põe no outro.

**2. O cliente foi reiniciado de verdade?** Os MCPs são carregados na
inicialização. Fechar a janela não basta: o processo segue vivo e só relê a
configuração ao subir. Encerre pela bandeja, ou o aplicativo inteiro.

**3. A entrada está mesmo no arquivo?** Leia o JSON e confirme. Vale checar se
existe mais de um arquivo de configuração no perfil — um antigo, esquecido, gera
horas de confusão.

**4. O `command` executa no seu sistema?** No Windows, `npx.cmd` como `command`
falha calado: `.cmd` não roda como processo. Use `cmd.exe /c npx …`.

**5. O servidor responde?** Rode a ponte à mão e leia a saída:

```bash
npx -y mcp-remote@latest https://mcp.mercadopago.com/mcp
```

Sucesso é `Connected to remote server` seguido de `Proxy established
successfully`. Qualquer parada antes disso mostra o erro real.

## Conversar com o servidor sem cliente nenhum

Quando é preciso saber o que o servidor responde, sem depender da interface:
suba a ponte, mande `initialize`, a notificação `notifications/initialized` e
então `tools/list` ou `tools/call`, um JSON por linha no stdin.

Dá para ler os schemas das ferramentas e chamar qualquer uma. É o caminho para
descobrir o parâmetro que falta quando a mensagem de erro é genérica.

Dê à ponte alguns segundos antes do `initialize`: na primeira execução o `npx`
baixa o pacote, e o `initialize` enviado cedo demais se perde.

## Erros conhecidos

| Erro | Causa | Conserto |
|---|---|---|
| `Input must be provided … when using --print` | o comando partiu de dentro de uma sessão do Claude Code | rodar em terminal comum, ou limpar `CLAUDECODE` e `CLAUDE_CODE_ENTRYPOINT` |
| `stdin isn't a terminal` no `mcp login` | o login OAuth é de mão dupla e precisa de terminal | usar `/mcp` no cliente, ou terminal interativo; a ponte também resolve, e grava o token em `~/.mcp-auth` |
| `ReferenceError: TransformStream is not defined` | Node abaixo de 20 | conferir `node -v` **no mesmo terminal** que roda o cliente |
| `command not found: npx` | npm antigo ou PATH sem o Node | npm 5.2+ e Node no PATH do terminal que inicia o cliente |
| Conecta e some depois | token expirado em `~/.mcp-auth` | apagar a pasta do servidor lá dentro e autenticar de novo |

## Fontes

- https://www.mercadopago.com.br/developers/en/docs/mcp-server/mcp-server-troubleshooting.md
- https://www.mercadopago.com.mx/developers/es/docs/mcp-server/connection.md
