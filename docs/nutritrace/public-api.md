# Public REST API

Plain JSON routes under `/api/v1` for your own scripts and automations, as an alternative to [MCP](mcp.md). It's pull-based: to be told the moment something happens instead of polling, an admin can set up outgoing webhooks (Settings, Webhooks). Off by default.

These routes (`/api/v1/diary`, `/api/v1/goals`, `/api/v1/meals`, `/api/v1/steps`, `/api/v1/body-composition`, `/api/v1/profile`) are separate from the [Federation API](federation-api.md) routes (`/api/v1/me`, `/api/v1/foods`, `/api/v1/workouts`, `/api/v1/activity`, `/api/v1/body-measurements`). Federation is a fixed contract for the sister Trace apps and is always on; the routes here are for personal automation and sit behind the switches below.

## Enable

```env
PUBLIC_API_ENABLED=1        # the read routes
PUBLIC_API_WRITE_ENABLED=1  # optional: the write routes too
```

Restart the server after changing them.

## Authentication

Create a token under **Settings, API Tokens** and send it as a bearer token. It needs a server with user accounts (a token needs an account to own it), and from the next release any signed-in user can make their own; earlier versions show API Tokens to admins only. Creating a token asks for your password, and your two-step code when that's on, and when it should expire. See [API tokens](../auth/sessions.md#api-tokens).

```
Authorization: Bearer nt_pat_...
```

The same `mcp:read` and `mcp:write` scopes cover both MCP and these routes: `mcp:read` reads through either, `mcp:write` writes through either. There's no separate REST scope. Every route works on the token owner's own data only.

## Read routes

Need `mcp:read`. Dates are `YYYY-MM-DD`.

| Method | Path | Notes |
|---|---|---|
| GET | `/api/v1/diary/:date` | One day's logged food items, and from the next release the day's completion marks: see [Day completion](#day-completion). |
| GET | `/api/v1/diary/:date/totals` | Summed nutrition (calories, macros, any micronutrients present) plus total water for the day. |
| GET | `/api/v1/goals` | Your current macro, micronutrient and water goals. A goal with `sharedGoal: false` has one target per weekday in `days`, starting on Sunday, and its `max`/`min` is the week's highest; `goals_by_weekday` gives those by named day and `today.targets` gives today's targets in your time zone. |
| GET | `/api/v1/meals/search?query=&limit=&include_recipes=` | Search your saved meals by name, or list them all when `query` is left out. Recipes are left out unless `include_recipes=true`. |
| GET | `/api/v1/meals/recent?limit=&include_recipes=&start=&end=` | Your most recently used saved meals. `start` and `end` (both optional) filter by the date each meal was last used. |
| GET | `/api/v1/meals/:id` | One saved meal with every item. |
| GET | `/api/v1/steps?start=&end=&source=` | Daily steps from your wellness data, one row per source. With neither date, the last 90 days; with one, the other side stays open. `source` picks one provider. Sources are never merged, and a missing day is missing, not 0. |
| GET | `/api/v1/body-composition?start=&end=&source=` | Body-composition readings from your wellness data, grouped by date and source, same date rules as steps. Readings from different sources stay separate, and a metric that wasn't recorded is left out. |
| GET | `/api/v1/profile` | Your gender and date of birth from your profile, each `null` when unset. |

## Write routes

Need `mcp:write` and `PUBLIC_API_WRITE_ENABLED=1`.

| Method | Path | Body | Notes |
|---|---|---|---|
| POST | `/api/v1/diary/:date/food` | `{food_id, meal?, quantity?, portion?, unit?, notes?}` | Adds one of your foods to a day. `food_id` comes from `GET /api/v1/foods` (scope `read:foods`) or the app. |
| POST | `/api/v1/diary/:date/water` | `{amount_ml, time?}` | Logs water. |
| POST | `/api/v1/diary/:date/meal` | `{meal_id, meal?}` | Adds a saved meal's items to a day. |
| PUT | `/api/v1/diary/:date/body-stat` | `{stats: {weight?, body_fat?, waist?, ...}}` | Merges the values into the day's body stats. |

Editing or deleting a diary entry and creating a food are MCP-only (`edit_diary_entry`, `delete_diary_entry`, `create_food`), behind `MCP_DESTROY_ENABLED`, the `mcp:destroy` scope and `confirm: true`. Food search lives in the federation route `GET /api/v1/foods?q=`.

## Day completion { #day-completion }

From the next release, `GET /api/v1/diary/:date` (and the MCP `list_diary_entries` tool) also returns the marks made with **Show Day Completion** (Settings, Diary), next to `date`, `items` and `count`. They're set in the app, not through this API.

| Field | Meaning |
|---|---|
| `completion_enabled` | Show Day Completion as saved on the server: `true`, `false`, or `null` when the server has no saved value (common after starting in single-user mode, where settings stay in the browser). Treat `null` as unknown, not as off. |
| `completed_at` | When the day was marked complete, in UTC (`YYYY-MM-DD HH:MM:SS`), or `null` while it's open. Marking a closed day again keeps the first time; reopening and closing it again gives a new one. |
| `completed_meals` | Meal slot numbers marked complete (the same numbers as each item's `meal`), sorted; `[]` when none. |

Any date can be marked complete, including past days and days with nothing logged (a fast). A `completed_at` stays even if the setting is later turned off. With `completion_enabled: true`, a `null` `completed_at` means the day is open; with `false`, completion isn't being tracked; with `null`, it's unknown which. Older servers leave all three fields out: treat a missing field as unknown.

```json
{ "date": "2026-09-12", "items": [], "count": 0,
  "completion_enabled": true, "completed_at": "2026-09-12 21:04:11", "completed_meals": [0, 2] }
```

## Limits and errors

Each token gets 60 requests a minute (`API_RATE_LIMIT_PER_MIN` to change it). Responses carry `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset`; a `429` also carries `Retry-After`.

A bad request (a malformed date, a date that doesn't exist such as `2026-02-31`, a reversed range, a meal id with no match) returns `400` with `{"error": "..."}`. A missing or invalid token returns `401`; a token without the needed scope returns `403`.

## Examples

```bash
# A day's logged food
curl -H "Authorization: Bearer nt_pat_..." \
  https://nutritrace.example.com/api/v1/diary/2026-09-12

# Log a food
curl -X POST -H "Authorization: Bearer nt_pat_..." -H "Content-Type: application/json" \
  -d '{"food_id": 42, "quantity": 1}' \
  https://nutritrace.example.com/api/v1/diary/2026-09-12/food

# Today's goal targets
curl -H "Authorization: Bearer nt_pat_..." \
  https://nutritrace.example.com/api/v1/goals

# Steps for a week, one row per source
curl -H "Authorization: Bearer nt_pat_..." \
  "https://nutritrace.example.com/api/v1/steps?start=2026-09-01&end=2026-09-07"
```
