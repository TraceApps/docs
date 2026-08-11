# MCP tool catalog

## What this page is

Every tool that NutriTrace exposes over the **Model Context Protocol**, in one table. This is the reference for external AI clients (Claude Desktop, Cursor, Codex, VS Code, custom agents) that connect to `/api/mcp`. The in-app Trace AI has its own separate tool set, catalogued in [Trace tool catalog](trace-tools.md).

Twelve tools across three phases, each phase gated by its own scope + env flag:

- **Read** (5 tools): `mcp:read` + `MCP_ENABLED=1`. Always available when MCP is on.
- **Write** (4 tools): `mcp:write` + `MCP_WRITE_ENABLED=1`. Additive log tools; everything they write shows up as normal entries in the diary UI.
- **Destructive** (3 tools): `mcp:destroy` + `MCP_DESTROY_ENABLED=1` + every call must include `confirm: true`.

Setup lives on the [Model Context Protocol](../nutritrace/mcp.md) page. This page is the tool reference; the setup page tells you how to turn it on.

## About the columns

- **Tool** is the exact `name` the MCP client sees; registered in `server/lib/mcp/tools/*.js`.
- **Purpose** is the one-line description an agent reads when deciding whether to call it.
- **Args** lists parameter names; `*` marks required. `date` defaults to today in the server's timezone.
- **Returns** describes the JSON shape wrapped in the MCP tool-response envelope.

## Read tools (`mcp:read`)

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `get_goals` | Macro / micro / water targets | none | `goals` object plus `water_goal_ml` |
| `get_daily_totals` | Summed nutrition + water for a day | `date` | `totals` (calories, macros, micros), `water_ml`, `item_count` |
| `list_diary_entries` | Raw item list from a diary day | `date` | `date`, `items[]`, `count` |
| `search_foods` | Text search over the user's local catalog | `query*`, `limit` | Match array (id, name, brand, barcode, portion, unit, nutrition) |
| `get_recent_foods` | Most-logged foods in the last 14 days | `limit` | Same shape as `search_foods` plus `last_logged_on` |
| `search_meals` | Search saved meals by name, or browse all when `query` is omitted | `query`, `limit`, `include_recipes` | Match array (id, name, is_recipe, servings, portion, unit, nutrition, favorite, usage_count, last_used_at). Recipes excluded by default. |
| `get_recent_meals` | Most-recently-used saved meals, ordered by `last_used_at` | `limit`, `include_recipes` | Same shape as `search_meals` |
| `get_meal_details` | Full contents of one saved meal or recipe (items[] + meta) | `meal_id*` | Meal meta plus `item_count` and `items[]` (name, portion, unit, quantity, per-item nutrition, source food id when known) |

## Write tools (`mcp:write`)

Additive; nothing is edited or deleted. Every write refuses if the target day is tombstoned (erased via the app), so an agent has to say so instead of silently resurrecting the day.

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `log_food` | Append a food from the user's catalog | `food_id*`, `date`, `meal`, `quantity`, `portion`, `unit`, `notes` | `logged` echo + `total_items_on_day`. Bumps `foods.usage_count` + `last_used_at`. |
| `log_water` | Add a water log entry (millilitres) | `amount_ml*`, `date`, `time` | `logged` echo + `total_ml_on_day` |
| `log_meal` | Log every item of a saved meal | `meal_id*`, `date`, `meal` | Item count + `total_items_on_day`. Recipes explicitly excluded (`is_recipe=1`). |
| `log_body_stat` | Set weight / body-fat / lengths on a day | `stats*` (object), `date` | Cleaned stats plus `current_stats`. Rejects untagged legacy body_stats (user must save once via the app to attach unit tags). |

## Destructive tools (`mcp:destroy` + `confirm: true`)

Belt-and-suspenders safety: the client already prompts per action (Claude Desktop, Cursor), and the tools *additionally* refuse without an explicit `confirm: true` in the arguments. A hallucinated call from a client that skips prompts still gets rejected.

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `delete_diary_entry` | Remove one item from a diary day | `entry_index*` (0-based, from `list_diary_entries`), `date`, `confirm*` | The removed entry (so the agent can offer to re-log it as undo) |
| `edit_diary_entry` | Patch one item's `quantity`, `portion`, `meal`, or `notes` | `entry_index*`, `date`, `patch*`, `confirm*` | `before` and `after` shapes. Portion change rescales nutrition. Refuses on recipe-splits and legacy flat-nutrition items. |
| `create_food` | Add a new food to the catalog | `name*`, `portion*`, `unit*`, `nutrition*`, `brand`, `category`, `barcode`, `notes`, `confirm*` | New food id + normalized fields. Rejects duplicates (case-insensitive name+brand). Caps per-nutriment values at 100000 to keep hallucinated numbers out of the catalog. |

## Scoping guarantees

Every tool query prepends `WHERE user_id = ?`. A token cannot read, write, or destroy anything outside its owner's account, even for admins. A static wiring test (`scripts/mcp-wiring.test.js`) fails CI if a future tool omits the scoping clause.

Tokens can hold any combination of scopes independently. Common configurations:

- **Read-only** (`mcp:read`): a "look at my data" token for read-mostly agents.
- **Read + write** (`mcp:read` + `mcp:write`): typical "help me log stuff" agent.
- **Full** (`mcp:read` + `mcp:write` + `mcp:destroy`): power-user token; every destructive call still needs `confirm: true`.

## Rate limits

Same per-token bucket as the Federation API: 60 requests per minute by default (`API_RATE_LIMIT_PER_MIN`). Responses include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (unix seconds). 429s carry `Retry-After`.

## Not currently exposed

Deliberately not in the current MCP surface:

- `delete_diary_day` (whole-day nuke). Blocked by tombstone-resurrection semantics; single-entry delete covers the common "I logged the wrong day" case safely.
- `log_recipe`. Recipes vary in yield semantics; logging component foods with `log_food` is the safe path today.
- `get_weight_history`, `list_meals`, `get_food_by_id`. Trivial to add; deferred until real demand.
- MCP prompts capability (guided workflows). Planned; no ETA.

## Related

- [Model Context Protocol setup](../nutritrace/mcp.md): turn it on, wire up Claude Desktop
- [Trace tool catalog](trace-tools.md): in-app Trace AI tools (separate mechanism)
- [Federation API](../nutritrace/federation-api.md): the older `/api/v1/*` REST surface that MCP shares its token model with
