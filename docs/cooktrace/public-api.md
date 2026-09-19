# Public REST API

Plain JSON routes under `/api/v1` for your own scripts and automations, as an alternative to [MCP](mcp.md). It's pull-based: to be told the moment something happens instead of polling, use [webhooks](webhooks.md). Off by default.

These routes (`/api/v1/cook-diary`, `/api/v1/shopping`, and the pantry stock route) are separate from the [NutriTrace federation](nt-federation.md) routes (`GET /api/v1/recipes`, `GET /api/v1/pantry`). Federation is a fixed contract for a sister Trace app and is always on; the routes here are for personal automation and sit behind the switches below.

## Enable

```env
PUBLIC_API_ENABLED=1        # the read routes
PUBLIC_API_WRITE_ENABLED=1  # optional: the write routes too
```

## Authentication

Create a token under **Settings, API Tokens** (admin, multi-user mode) and send it as a bearer token:

```
Authorization: Bearer ct_pat_...
```

The same `mcp:read` and `mcp:write` scopes cover both MCP and these routes: `mcp:read` reads through either, `mcp:write` writes through either. There's no separate REST scope.

## Read routes

Need `mcp:read`.

| Method | Path | Notes |
|---|---|---|
| GET | `/api/v1/cook-diary?date_from=&date_to=&kind=&limit=` | Cook Diary entries, newest first. Optionally limited to a date range (`YYYY-MM-DD`) or a `kind` (`cooked` or `planned`). |
| GET | `/api/v1/shopping?include_checked=` | The shopping list. Unchecked items only unless `include_checked=true`. |

## Write routes

Need `mcp:write` and `PUBLIC_API_WRITE_ENABLED=1`.

| Method | Path | Body | Notes |
|---|---|---|---|
| POST | `/api/v1/cook-diary` | `{ recipe_id, date?, servings?, notes?, meal_type?, rating? }` | Logs that you cooked a recipe. Fires the `meal.cooked` webhook. |
| PATCH | `/api/v1/shopping/:id/check` | `{ checked }` | Checks or unchecks an item. Checking the last one fires `shopping_list.completed`. |
| PATCH | `/api/v1/pantry/:id/stock` | `{ in_stock?, quantity? }` | Updates an existing pantry item's stock or quantity. Doesn't create items. Going out of stock fires `pantry.out_of_stock`. |

Creating recipes or pantry items, adding shopping items, and deleting entries aren't exposed here. Those are available through MCP's destructive tier, which needs `MCP_DESTROY_ENABLED=1`, the `mcp:destroy` scope, and `confirm: true` on every call.

## Sister apps: the `shopping` scope

A token with the `shopping` scope gets its own version of `/api/v1/shopping`. It's always on, needs no `PUBLIC_API_*` or MCP switch, and reaches nothing but the token owner's shopping list. [NoteTrace](../notetrace/cooktrace.md) uses it to send items and show the list. A token without `shopping` sees the routes above as normal.

| Method | Path | Body | Returns |
|---|---|---|---|
| GET | `/api/v1/shopping?include_checked=` | | `{ count, items }`, sorted like the Shopping page. Checked items are included unless `include_checked=false`. |
| POST | `/api/v1/shopping` | `{ items: [{ name, quantity?, unit?, aisle? }] }` or a single item | `201 { added, skipped }`. Names are title-cased like the app, a name matching a pantry item takes its aisle, and an item already on the list unchecked is skipped. Up to 200 at a time. |
| PATCH | `/api/v1/shopping/:id/check` | `{ checked }` | `{ ok, item_id, name, checked }` |
| DELETE | `/api/v1/shopping/checked` | | `{ removed }`: clears checked items. |

## Limits and errors

Each token gets 60 requests a minute (`API_RATE_LIMIT_PER_MIN` to change it). Responses carry `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset`; a `429` also carries `Retry-After`.

A bad request returns `400` with `{ "error": "..." }`. A missing or invalid token returns `401`, and a token without the needed scope returns `403`. A route whose switch is off returns `404`.

## Examples

```bash
# Recent Cook Diary entries
curl -H "Authorization: Bearer ct_pat_..." \
  https://cooktrace.example.com/api/v1/cook-diary

# Log a cook
curl -X POST -H "Authorization: Bearer ct_pat_..." -H "Content-Type: application/json" \
  -d '{"recipe_id": 42, "servings": 4}' \
  https://cooktrace.example.com/api/v1/cook-diary

# Mark a pantry item out of stock
curl -X PATCH -H "Authorization: Bearer ct_pat_..." -H "Content-Type: application/json" \
  -d '{"in_stock": false}' \
  https://cooktrace.example.com/api/v1/pantry/17/stock
```
