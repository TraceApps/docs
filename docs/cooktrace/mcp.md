# Model Context Protocol (MCP)

CookTrace speaks the [Model Context Protocol](https://modelcontextprotocol.io) so any MCP-compatible AI client (Claude Desktop, Cursor, Codex, VS Code, custom agents) can read your recipes, pantry, shopping list, and cook diary directly from your self-hosted instance.

This is different from **Trace AI** (which runs inside the app). MCP lets an external agent, running on your own machine, ask questions of your CookTrace server without any of your data going through CookTrace's own AI provider settings.

Off by default. Opt in with one env var + one API token.

## Available tools

Fourteen tools across three tiers. Setup for each tier below; full arg/return reference in the [MCP tool catalog](../reference/mcp-tools.md).

### Read (always available when MCP is on)

Six read-only tools: `search_recipes`, `get_recipe`, `recent_recipes`, `list_pantry`, `list_shopping_list`, `list_cook_diary`.

### Write (Phase 2)

Four additive tools: `log_cook`, `add_shopping_item`, `check_shopping_item`, `update_pantry_stock`. Everything they write shows up as normal entries in the app's UI, editable and deletable through the app like anything else. Off by default; turn on with `MCP_WRITE_ENABLED=1` AND a token that holds `mcp:write`. Either missing and the write tools simply don't appear in `tools/list`.

### Destructive (Phase 3)

Four destructive tools: `create_recipe`, `add_pantry_item`, `delete_cook_diary_entry`, `remove_shopping_item`. Off by default. Three gates ALL required:

1. `MCP_DESTROY_ENABLED=1` on the server.
2. Token holds `mcp:destroy` (mint separately; a `mcp:write` token does NOT unlock destroy).
3. Every call includes `confirm: true` in the arguments. Belt-and-suspenders: your MCP client already prompts per action, but the tool refuses without the arg so a hallucinated call from a client that skips prompts still gets rejected.

## Enable it

Add one env var to your `docker-compose.yml`:

```yaml
services:
  cooktrace:
    environment:
      - MCP_ENABLED=1
```

Redeploy. The endpoint at `/api/mcp` starts responding to authenticated MCP clients. Everything else remains untouched; existing users see no change.

## Mint an MCP token

Log in as an admin on a **multi-user** instance (API tokens need a real account to own them, and single-user mode doesn't have one, so this section is hidden there), then:

1. Open **Settings → API Tokens**
2. **New token** → name it (e.g. "claude-desktop")
3. Check the `mcp:read` scope
4. Create

The raw token (`ct_pat_...`) is shown once. Copy it now; the server only keeps a SHA-256 hash after this.

## Wire up Claude Desktop

Claude Desktop reads MCP servers from a JSON config file:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`

Add a `mcpServers` entry:

```json
{
  "mcpServers": {
    "cooktrace": {
      "url": "https://cooktrace.example.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ct_pat_your_token_here"
      }
    }
  }
}
```

Restart Claude Desktop. In a new chat, ask "What can I cook with what's in my pantry?" or "Add milk and eggs to my shopping list" and Claude will call the CookTrace tools directly.

## Wire up other clients

The transport is standard MCP Streamable HTTP (single POST endpoint). Any client that accepts a base URL + bearer header works:

- **Cursor**: Settings → MCP → Add server with the same URL and Authorization header.
- **Codex / OpenAI Assistants**: Point at the URL; pass the bearer as a header.
- **Custom** (Node / Python): Use the official `@modelcontextprotocol/sdk` client. The endpoint speaks JSON-RPC 2.0 framed over Streamable HTTP.

## Security model

- **Off by default.** Nothing exposed without `MCP_ENABLED=1`.
- **Scoped tokens.** `mcp:read` is a distinct scope from `mcp:write` / `mcp:destroy`.
- **Rate-limited per token.** Overspend returns 429.
- **Per-user isolation.** Every tool query is scoped to the token owner's data; no cross-user leakage is possible, even for admins. `log_cook` is the one intentional exception: it can log a cook against a recipe someone else shared with your Kitchen, exactly like the app's own "I cooked this" button already allows.
- **Origin gating (DNS-rebinding defense).** Server-to-server clients (Claude Desktop, Cursor, etc.) send no Origin header and pass. Browser-based MCP inspectors need to be listed in `ALLOWED_ORIGINS` explicitly. This is the [MCP spec's recommended](https://modelcontextprotocol.io/specification/basic/authorization) defense against DNS rebinding attacks and CookTrace does not soften it with a "trust the Host header" fallback.
- **Bearer over HTTPS only.** Never send the token over plain HTTP; it's equivalent to a user password for the tools it grants.
- **Recipe creation fans out to your Kitchen the same way the app does.** `create_recipe` calls the same auto-share hook `POST /api/recipes` uses, so an MCP-created recipe reaches Kitchen members identically to one created in the app.

## Env vars

| Variable | Default | Description |
|---|---|---|
| `MCP_ENABLED` | `0` | Set to `1` to expose `/api/mcp`. |
| `MCP_WRITE_ENABLED` | `0` | Set to `1` to allow write tools (`log_cook`, `add_shopping_item`, `check_shopping_item`, `update_pantry_stock`) to be registered. Also requires the calling token to hold `mcp:write`. |
| `MCP_DESTROY_ENABLED` | `0` | Set to `1` to allow the destructive tools (`create_recipe`, `add_pantry_item`, `delete_cook_diary_entry`, `remove_shopping_item`) to be registered. Also requires the calling token to hold `mcp:destroy` AND every call to include `confirm: true`. |
| `ALLOWED_ORIGINS` | (empty) | Comma-separated list of origins that browser-based MCP clients may use. Server-to-server clients (no Origin header) always pass. Leave empty unless you're specifically using the MCP Inspector in a browser. |

## Verifying

A smoke-test script lives at `scripts/mcp-smoke.mjs` in the [CookTrace repo](https://github.com/traceapps/cooktrace):

```bash
git clone https://github.com/traceapps/cooktrace && cd cooktrace
node scripts/mcp-smoke.mjs https://cooktrace.example.com ct_pat_your_token_here
```

Add `--writes` to also exercise the write tools (needs `mcp:write` + `MCP_WRITE_ENABLED=1`); the script looks up a real recipe from your own library via `search_recipes` before calling `log_cook`, so it never guesses an id. Add `--destroy` on top to verify the destructive tools' gate-refusal path (needs `mcp:destroy` + `MCP_DESTROY_ENABLED=1`; the smoke script never actually deletes or creates anything destructive, it only checks that a call without `confirm: true` is rejected).

## Troubleshooting

**`{"error":"MCP not enabled on this server"}` (404).** `MCP_ENABLED=1` isn't set in the container env. Redeploy after adding it.

**`{"error":"auth_missing"}` (401).** No `Authorization: Bearer ct_pat_...` header.

**`{"error":"auth_scope"}` (403).** The token was minted without `mcp:read`. Revoke and create a new one with the scope checked.

**`{"error":"Origin not allowed"}` (403).** A browser-based client is sending an `Origin` header that isn't in `ALLOWED_ORIGINS`. Server-to-server clients (Claude Desktop, Cursor CLI) don't send Origin and are unaffected.

**429 rate_limited.** Backoff per the `Retry-After` header.

**Settings → API Tokens isn't there.** The section only appears on a multi-user instance with a signed-in admin: a token needs a real account to own it, and single-user mode has none. Enable user management first.

## Roadmap

**Later**: MCP prompts capability (guided workflows), additional read tools (cookbooks, kitchens) as demand justifies them, and possibly `edit_recipe` / `edit_cook_diary_entry` (currently out of scope, the existing tools cover the primary "log what I just did" and "add what I need" use cases).
