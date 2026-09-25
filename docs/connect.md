# Connecting

The address is the same everywhere:

```
https://api.glockapps.com/mcp
```

Two ways to authenticate, and the client decides which one you get:

- **OAuth** — the client opens a GlockApps page, you paste your API key there once, and the client
  is given a token scoped to this connector. Claude and ChatGPT do this.
- **API key header** — the client sends your key on every request. For clients with no OAuth
  support. Anything that can read the file it is configured from can read the key, so treat that
  file the way you treat any other secret.

Your key is on [Account → API / MCP](https://app.glockapps.com/account/api). API access has to be
enabled on your plan.

---

## Claude.ai and Claude Desktop

1. **Settings → Connectors → Add custom connector**.
2. Paste `https://api.glockapps.com/mcp`. Leave the optional Client ID and Client Secret empty.
3. **Connect**. A GlockApps page opens.
4. Paste your API key. That is the only time you need it.

The connector then appears in the tools menu of any chat.

## Claude Code

```bash
claude mcp add --transport http glockapps https://api.glockapps.com/mcp
```

Run `/mcp` in the session and pick **Authenticate**; a browser window completes the OAuth flow.
`claude mcp list` shows it as connected afterwards.

## ChatGPT

Requires developer mode while the connector is not yet listed in the ChatGPT directory.

1. **Settings → Security and login → Developer mode**: turn it on. (Settings → Plugins → Developer
   mode leads to the same switch.)
2. Open **Plugins** in the sidebar, select **+** → **Create app**, then **Create MCP App**.
3. Name it GlockApps. Under **Connection** keep **Server URL** and paste
   `https://api.glockapps.com/mcp`.
4. Leave **Authentication** on **OAuth**. ChatGPT discovers the rest by itself: **Advanced OAuth
   settings** shows Dynamic Client Registration and the `inbox_placement:read` /
   `inbox_placement:write` scopes, and nothing there needs changing.
5. Tick **I understand and want to continue**, select **Create**, then paste your API key on the
   GlockApps page that opens.

## Cursor

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` inside one:

```json
{
    "mcpServers": {
        "glockapps": {
            "url": "https://api.glockapps.com/mcp",
            "headers": { "Authorization": "Bearer YOUR_GLOCKAPPS_API_KEY" }
        }
    }
}
```

Settings → MCP shows the server and its tools once the file is saved.

## Windsurf

`~/.codeium/windsurf/mcp_config.json`:

```json
{
    "mcpServers": {
        "glockapps": {
            "serverUrl": "https://api.glockapps.com/mcp",
            "headers": { "Authorization": "Bearer YOUR_GLOCKAPPS_API_KEY" }
        }
    }
}
```

## Continue

In `~/.continue/config.yaml`:

```yaml
mcpServers:
    - name: glockapps
      type: streamable-http
      url: https://api.glockapps.com/mcp
      requestOptions:
          headers:
              Authorization: Bearer YOUR_GLOCKAPPS_API_KEY
```

## Any other MCP client

Streamable HTTP, protocol revision 2025-06-18 or later. Point it at
`https://api.glockapps.com/mcp` and either let it discover OAuth — the server publishes
`/.well-known/oauth-protected-resource/mcp` and `/.well-known/oauth-authorization-server`, with
dynamic client registration and PKCE — or send `Authorization: Bearer <your API key>`.

## Checking it works

Ask for something read-only first:

> What is my GlockApps test credit balance?

That calls `get_balance` and spends nothing. If it answers, the connection is good.

To inspect the wire directly:

```bash
npx @modelcontextprotocol/inspector
```

Connect it to the same URL with your API key as a bearer header, and the tool list should show
fifteen tools.
