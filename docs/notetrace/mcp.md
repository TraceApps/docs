# Model Context Protocol (MCP)

NoteTrace speaks the [Model Context Protocol](https://modelcontextprotocol.io), so an MCP client (Claude Desktop, Cursor, Codex, VS Code, your own agent) can search, read, and update your notes on your own server.

This is separate from [Trace in NoteTrace](trace.md), which runs inside the app. The tools are the same ones Trace uses, with the same rules, so an agent can do what Trace can and nothing more.

Off by default. Opt in with an env var and an API token.

## Available tools

Fourteen tools across three tiers. The full argument reference is in the [MCP tool catalog](../reference/mcp-tools.md#notetrace).

- **Read** (`mcp:read` and `MCP_ENABLED=1`): `search_notes`, `get_note`, `list_labels`, `list_reminders`, `list_tasks`.
- **Write** (`mcp:write` and `MCP_WRITE_ENABLED=1`): `create_note`, `update_note`, `append_to_note`, `add_checklist_items`, `check_checklist_item`, `set_due_date`, `set_reminder`, `set_labels`. A text rewrite from `update_note` keeps the previous text in version history. `set_reminder` reads a time without an offset in its `time_zone` argument, or else in the time zone of your most recent reminder set from a device, or else the server's.
- **Destructive** (`mcp:destroy`, `MCP_DESTROY_ENABLED=1`, and `confirm: true` on every call): `move_to_trash`. Trashed notes can still be restored for 30 days.

A tool from a tier that isn't enabled, or that the token's scopes don't cover, doesn't appear in `tools/list`.

## Enable it

```yaml
services:
  notetrace:
    environment:
      - MCP_ENABLED=1
      # Optional tiers:
      # - MCP_WRITE_ENABLED=1
      # - MCP_DESTROY_ENABLED=1
```

Redeploy. The endpoint is `/api/mcp`.

## Create a token

API tokens need an account to belong to, so they're available on a server with user accounts (not single-user mode). As an admin:

1. Open **Settings, API Tokens**.
2. Create a token, name it (for example "claude-desktop"), and tick the scopes: `mcp:read`, plus `mcp:write` and `mcp:destroy` if you want those tiers.
3. Copy the token (it starts with `note_pat_`). It's shown once.

The token acts as the account that created it and sees only that account's notes, plus notes shared with it.

## Connect a client

Claude Desktop reads `claude_desktop_config.json` (`~/Library/Application Support/Claude/` on macOS, `%APPDATA%\Claude\` on Windows, `~/.config/Claude/` on Linux):

```json
{
  "mcpServers": {
    "notetrace": {
      "url": "https://notes.example.com/api/mcp",
      "headers": { "Authorization": "Bearer note_pat_your_token_here" }
    }
  }
}
```

Restart Claude Desktop and ask "What's on my groceries list?" or "Add batteries to my hardware store list". Any client that takes a URL and a bearer header works the same way; the transport is MCP Streamable HTTP in stateless mode (one `POST` endpoint).

## Security model

- **Off by default**, and each tier has its own env flag and token scope.
- **Same rules as the app.** View-only shared notes can't be changed, and only a note's owner can set its reminder or trash it.
- **Rate limited per token** (`API_RATE_LIMIT_PER_MIN`, 60 by default). Over the limit returns `429`.
- **Origin check.** Server-to-server clients send no `Origin` header and pass. A browser-based MCP client must be listed in `ALLOWED_ORIGINS`.
- **Use HTTPS.** The token is as powerful as the account for the scopes it holds.

## Env vars

| Variable | Default | Description |
|---|---|---|
| `MCP_ENABLED` | unset | `1` exposes `/api/mcp` with the read tools. |
| `MCP_WRITE_ENABLED` | unset | `1` adds the write tools, for tokens with `mcp:write`. |
| `MCP_DESTROY_ENABLED` | unset | `1` adds `move_to_trash`, for tokens with `mcp:destroy`; every call needs `confirm: true`. |
| `ALLOWED_ORIGINS` | unset | Comma-separated origins allowed for browser-based MCP clients. |

## Troubleshooting

**404 "MCP not enabled on this server".** `MCP_ENABLED=1` isn't set in the container.

**401.** The `Authorization: Bearer note_pat_...` header is missing or the token was revoked.

**403 on the endpoint.** The token has none of the `mcp:*` scopes.

**A write tool is missing from the list.** Both the server flag and the token scope are needed for its tier.

**Settings, API Tokens isn't there.** It only appears for an admin on a server with user accounts.

## Related

- [MCP tool catalog](../reference/mcp-tools.md#notetrace)
- [Trace in NoteTrace](trace.md)
- [Webhooks](webhooks.md)
