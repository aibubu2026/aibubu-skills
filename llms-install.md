# Aibubu remote MCP setup for Cline

Aibubu hosts a remote MCP server (Streamable HTTP). No local server, build step or package install is needed.

| Region | Account site | MCP URL |
| --- | --- | --- |
| International (default) | aibubu.ai | `https://aibubu.ai/mcp` |
| China | aibubu.cn | `https://aibubu.cn/mcp` |

Use the international URL unless the user says they use aibubu.cn. Ask if unsure. Use the server name `aibubu` in both cases.

## Add the server

Open the MCP Servers panel, go to the Remote Servers tab, and enter the name `aibubu`, the URL above and the transport **Streamable HTTP**.

Or edit the MCP settings file (Configure MCP Servers in the IDE extension; `~/.cline/data/settings/cline_mcp_settings.json`, or the path in `CLINE_MCP_SETTINGS_PATH`, for the CLI). Merge this entry into the existing `mcpServers` object, keep other servers untouched, and update an existing `aibubu` entry instead of adding a second one:

```json
{
  "mcpServers": {
    "aibubu": {
      "type": "streamableHttp",
      "url": "https://aibubu.ai/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Always set `type` to `streamableHttp`: when it is omitted Cline falls back to the legacy SSE transport, which this server does not use. Keep `autoApprove` empty: Aibubu generation tools spend credits.

With Cline CLI:

```sh
cline mcp add aibubu https://aibubu.ai/mcp --transport http --yes
```

## Sign in

The recommended method is account sign-in (OAuth). When Cline shows the server as needing authentication, choose Authenticate and let the user sign in to Aibubu in the browser and click Allow. Do not ask the user to paste passwords or tokens into the chat. In Cline CLI, run `cline mcp`, choose Authorize OAuth, then select `aibubu` (needs an interactive terminal; stop and let the user finish it).

Alternative for clients or setups without browser sign-in: create an API key on the Aibubu API Keys page, store it in an environment variable, and add a header to the same entry. Never write the key into project files:

```json
"headers": { "Authorization": "Bearer sk_aibubu_your_api_key" }
```

or `cline mcp add aibubu https://aibubu.ai/mcp --transport http --header "Authorization: Bearer $AIBUBU_API_KEY" --yes`.

## Verify

Confirm the server shows as connected, then ask for a real free `get_balance` call and a `list_models` call. Tool discovery alone is not proof of authorization. Do not call `generate_image`, `generate_video`, `generate_batch` or `upscale_image` during installation.

Before any paid generation, call `quote` and `get_balance`, tell the user the estimated credits, and get explicit confirmation. If authentication fails, report the client version and the redacted error; do not disable certificate checks or enable blanket auto-approval to get around it.

## Optional skill

The `aibubu-models` Agent Skill (REST API with `curl`, uses `AIBUBU_API_KEY`) is separate from MCP: `npx skills add aibubu2026/aibubu-skills`.

Docs: https://docs.aibubu.ai/en/agent-tools (China: https://docs.aibubu.cn/cn/agent-tools)
