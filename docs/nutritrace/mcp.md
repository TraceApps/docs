# Model Context Protocol (MCP)

NutriTrace speaks the [Model Context Protocol](https://modelcontextprotocol.io) so any MCP-compatible AI client (Claude Desktop, Cursor, Codex, VS Code, custom agents) can read your food catalog, diary, and goals directly from your self-hosted instance.

This is different from **Trace AI** (which runs inside the app with 16 built-in tools). MCP lets an external agent, running on your own machine, ask questions of your NutriTrace server without any of your data going through NutriTrace's own AI provider settings.

Off by default. Opt in with one env var + one API token.

## Available tools

### Read (Phase 1)

Five tools, all read-only, all scoped to the user the token belongs to:

| Tool | Returns |
|---|---|
| `get_goals` | Your macro / micronutrient / water goal targets |
| `get_daily_totals` | Summed calories + macros + water for a day |
| `list_diary_entries` | Raw item list from a day's diary |
| `search_foods` | Text search over your local foods catalog |
| `get_recent_foods` | Most-recently-logged foods (last 14 days) |

### Write (Phase 2)

Four additive write tools. All show up as normal entries in the diary UI, editable and deletable through the app like anything else.

| Tool | Effect |
|---|---|
| `log_food` | Append a food from your catalog to a diary day (needs `food_id` from `search_foods`). |
| `log_water` | Append a water log entry (millilitres). |
| `log_meal` | Log every item of a saved meal (recipes deliberately excluded). |
| `log_body_stat` | Set weight / body fat / waist / etc. on a diary day (merges with existing stats). |

Write tools are off by default. Turn them on with `MCP_WRITE_ENABLED=1` on the server AND mint a token that holds `mcp:write` in addition to `mcp:read`. Either missing and the write tools simply don't appear in `tools/list`; an agent can't attempt them.

### Destructive (Phase 3)

Three destructive tools. Removing or mutating pre-existing data on the server.

| Tool | Effect |
|---|---|
| `delete_diary_entry` | Remove one item from a diary day by 0-based `entry_index` (from `list_diary_entries`). |
| `edit_diary_entry` | Patch one item's `quantity`, `portion`, `meal` slot, or `notes`. Portion change re-scales nutrition. |
| `create_food` | Add a new food to your catalog (rejects duplicates by name+brand). |

Off by default. Three gates all required:

1. `MCP_DESTROY_ENABLED=1` on the server.
2. Token holds `mcp:destroy` (mint separately; a `mcp:write` token does NOT unlock destroy).
3. Every call includes `confirm: true` in the arguments. Belt-and-suspenders: your MCP client already prompts per action, but the tool refuses without the arg so a hallucinated call from a client that skips prompts still gets rejected.

Delete tools return the removed content in the response body so the agent can offer to re-log it as a form of undo. Edit tools return `before` + `after` so the agent can offer to revert.

## Enable it

Add one env var to your `docker-compose.yml`:

```yaml
services:
  nutritrace:
    environment:
      - MCP_ENABLED=1
```

Redeploy. The endpoint at `/api/mcp` starts responding to authenticated MCP clients. Everything else remains untouched; existing users see no change.

## Mint an MCP token

Log in as an admin, then:

1. Open **Settings → API Tokens**
2. **New token** → name it (e.g. "claude-desktop")
3. Check the `mcp:read` scope
4. Create

The raw token (`nt_pat_...`) is shown once. Copy it now; the server only keeps a SHA-256 hash after this.

## Wire up Claude Desktop

Claude Desktop reads MCP servers from a JSON config file:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`

Add a `mcpServers` entry:

```json
{
  "mcpServers": {
    "nutritrace": {
      "url": "https://nutritrace.example.com/api/mcp",
      "headers": {
        "Authorization": "Bearer nt_pat_your_token_here"
      }
    }
  }
}
```

Restart Claude Desktop. In a new chat, ask "What did I eat today?" or "What's my calorie goal?" and Claude will call the NutriTrace tools directly.

## Wire up other clients

The transport is standard MCP Streamable HTTP (single POST endpoint). Any client that accepts a base URL + bearer header works:

- **Cursor**: Settings → MCP → Add server with the same URL and Authorization header.
- **Codex / OpenAI Assistants**: Point at the URL; pass the bearer as a header.
- **Custom** (Node / Python): Use the official `@modelcontextprotocol/sdk` client. The endpoint speaks JSON-RPC 2.0 framed over Streamable HTTP.

## Security model

- **Off by default.** Nothing exposed without `MCP_ENABLED=1`.
- **Scoped tokens.** `mcp:read` is a distinct scope; a token minted for `read:foods` (federation) does not unlock MCP, and vice-versa.
- **Rate-limited per token.** Same bucket as the rest of the API tokens; overspend returns 429.
- **Per-user isolation.** Every tool query is scoped to the token owner's data; no cross-user leakage is possible, even for admins.
- **Origin gating (DNS-rebinding defense).** Server-to-server clients (Claude Desktop, Cursor, etc.) send no Origin header and pass. Browser-based MCP inspectors need to be listed in `ALLOWED_ORIGINS` explicitly. This is the [MCP spec's recommended](https://modelcontextprotocol.io/specification/basic/authorization) defense against DNS rebinding attacks and NutriTrace does not soften it with a "trust the Host header" fallback.
- **Bearer over HTTPS only.** Never send the token over plain HTTP; it's equivalent to a user password for the tools it grants.

## Env vars

| Variable | Default | Description |
|---|---|---|
| `MCP_ENABLED` | `0` | Set to `1` to expose `/api/mcp`. |
| `MCP_WRITE_ENABLED` | `0` | Set to `1` to allow write tools (`log_food`, `log_water`, `log_meal`, `log_body_stat`) to be registered. Also requires the calling token to hold `mcp:write`. |
| `MCP_DESTROY_ENABLED` | `0` | Set to `1` to allow destructive tools (`delete_diary_entry`, `edit_diary_entry`, `create_food`) to be registered. Also requires the calling token to hold `mcp:destroy` AND every call to include `confirm: true`. |
| `ALLOWED_ORIGINS` | (empty) | Comma-separated list of origins that browser-based MCP clients may use. Server-to-server clients (no Origin header) always pass. Leave empty unless you're specifically using the MCP Inspector in a browser. |

## Verifying

A smoke-test script lives at `scripts/mcp-smoke.mjs` in the [NutriTrace repo](https://github.com/traceapps/nutritrace). Handshake, list tools, invoke each one, and verify negative-path (missing bearer, bad origin) gating, all in one command:

```bash
git clone https://github.com/traceapps/nutritrace && cd nutritrace
node scripts/mcp-smoke.mjs https://nutritrace.example.com nt_pat_your_token_here
```

Expected output ends with `9 passed, 0 failed`. Add `--writes` to also exercise the write tools (needs `mcp:write` + `MCP_WRITE_ENABLED=1`), and `--destroy` on top to verify the destructive tools' gate-refusal paths (needs `mcp:destroy` + `MCP_DESTROY_ENABLED=1`; the smoke script never actually deletes anything).

## Troubleshooting

**`{"error":"MCP not enabled on this server"}` (404).** `MCP_ENABLED=1` isn't set in the container env. Redeploy after adding it.

**`{"error":"auth_missing"}` (401).** No `Authorization: Bearer nt_pat_...` header.

**`{"error":"auth_scope"}` (403).** The token was minted without `mcp:read`. Revoke and create a new one with the scope checked.

**`{"error":"Origin not allowed"}` (403).** A browser-based client is sending an `Origin` header that isn't in `ALLOWED_ORIGINS`. Server-to-server clients (Claude Desktop, Cursor CLI) don't send Origin and are unaffected.

**429 rate_limited.** Backoff per the `Retry-After` header. Same per-token rate cap as `/api/v1/*`.

## Roadmap

**Later**: MCP prompts capability (guided workflows), additional read tools (weight history, body composition, recipes) as demand justifies them, and possibly `delete_diary_day` (currently blocked by tombstone-resurrection semantics; single-entry delete is the safer starting point).

Feature request and progress: [issue #103](https://github.com/traceapps/nutritrace/issues/103) (thanks @javydekoning).
