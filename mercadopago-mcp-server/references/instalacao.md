# Instalação por cliente

## Claude Code

```bash
claude mcp add --scope user mercadopago -- npx -y mcp-remote@latest https://mcp.mercadopago.com/mcp
```

`--scope user` vale para todos os projetos; `--scope local` só para o projeto
atual. Conferir com `claude mcp list` — cada servidor sai com o estado
(`Connected` ou `Needs authentication`).

Se o comando responder **"Input must be provided either through stdin or as a
prompt argument when using --print"**, ele foi disparado de dentro de uma sessão
do próprio Claude Code: o processo filho herda as variáveis da sessão e entra em
modo não-interativo. Rode num terminal comum, ou limpe `CLAUDECODE` e
`CLAUDE_CODE_ENTRYPOINT` antes.

## Claude Desktop

`claude_desktop_config.json` aceita **apenas** servidor de processo local. O
endpoint HTTP entra pela ponte:

```json
{
  "mcpServers": {
    "mercadopago": {
      "command": "C:\Windows\System32\cmd.exe",
      "args": ["/c", "npx", "-y", "mcp-remote@latest", "https://mcp.mercadopago.com/mcp"]
    }
  }
}
```

No Windows, apontar `command` direto para `npx.cmd` falha em silêncio: o sistema
não executa `.cmd` como processo. Use `cmd.exe /c`.

Localização do arquivo:

| Sistema | Caminho |
|---|---|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |

Esse arquivo guarda também preferências da aplicação. Faça backup, edite só o
bloco `mcpServers` e valide o JSON antes de reiniciar — arquivo quebrado derruba
as configurações junto.

## Outros clientes

Cursor, VS Code, Windsurf e Cline usam `mcp.json` ou `.cursor/mcp.json`, com o
mesmo formato de processo local. A documentação oficial também descreve passar
um access token por header, como alternativa ao OAuth:

```bash
npx -y mcp-remote@latest https://mcp.mercadopago.com/mcp --header 'Authorization:Bearer <ACCESS_TOKEN>'
```

Nesse caminho o token fica no arquivo de configuração. Em máquina compartilhada
ou repositório versionado, o OAuth é preferível: a credencial vai para
`~/.mcp-auth`, fora do projeto.

## Fontes

- https://www.mercadopago.com.br/developers/pt/docs/mcp-server/overview
- https://www.mercadopago.com.mx/developers/es/docs/mcp-server/connection.md
- https://www.mercadopago.com.br/developers/en/docs/mcp-server/mcp-server-troubleshooting.md
