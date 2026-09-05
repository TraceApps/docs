# MCP tool catalog

## What this page is

Every tool that NutriTrace and LiftTrace expose over the **Model Context Protocol**, in one table per app. This is the reference for external AI clients (Claude Desktop, Cursor, Codex, VS Code, custom agents) that connect to each app's `/api/mcp`. The in-app Trace AI has its own separate tool set, catalogued in [Trace tool catalog](trace-tools.md).

Both apps split their MCP surface into the same three phases, each gated by its own scope + env flag:

- **Read**: `mcp:read` + `MCP_ENABLED=1`. Always available when MCP is on.
- **Write**: `mcp:write` + `MCP_WRITE_ENABLED=1`. Additive log tools; everything they write shows up as normal entries in the app's own UI.
- **Destructive**: `mcp:destroy` + `MCP_DESTROY_ENABLED=1` + every call must include `confirm: true`.

Setup lives on each app's own MCP page: [NutriTrace](../nutritrace/mcp.md), [LiftTrace](../lifttrace/mcp.md). This page is the tool reference; the setup pages tell you how to turn it on.

## About the columns

- **Tool** is the exact `name` the MCP client sees; registered in `server/lib/mcp/tools/*.js`.
- **Purpose** is the one-line description an agent reads when deciding whether to call it.
- **Args** lists parameter names; `*` marks required. `date` defaults to today in the server's timezone.
- **Returns** describes the JSON shape wrapped in the MCP tool-response envelope.

## NutriTrace

Twelve tools across three phases.

### Read tools (`mcp:read`)

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

### Write tools (`mcp:write`)

Additive; nothing is edited or deleted. Every write refuses if the target day is tombstoned (erased via the app), so an agent has to say so instead of silently resurrecting the day.

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `log_food` | Append a food from the user's catalog | `food_id*`, `date`, `meal`, `quantity`, `portion`, `unit`, `notes` | `logged` echo + `total_items_on_day`. Bumps `foods.usage_count` + `last_used_at`. |
| `log_water` | Add a water log entry (millilitres) | `amount_ml*`, `date`, `time` | `logged` echo + `total_ml_on_day` |
| `log_meal` | Log every item of a saved meal | `meal_id*`, `date`, `meal` | Item count + `total_items_on_day`. Recipes explicitly excluded (`is_recipe=1`). |
| `log_body_stat` | Set weight / body-fat / lengths on a day | `stats*` (object), `date` | Cleaned stats plus `current_stats`. Rejects untagged legacy body_stats (user must save once via the app to attach unit tags). |

### Destructive tools (`mcp:destroy` + `confirm: true`)

Belt-and-suspenders safety: the client already prompts per action (Claude Desktop, Cursor), and the tools *additionally* refuse without an explicit `confirm: true` in the arguments. A hallucinated call from a client that skips prompts still gets rejected.

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `delete_diary_entry` | Remove one item from a diary day | `entry_index*` (0-based, from `list_diary_entries`), `date`, `confirm*` | The removed entry (so the agent can offer to re-log it as undo) |
| `edit_diary_entry` | Patch one item's `quantity`, `portion`, `meal`, or `notes` | `entry_index*`, `date`, `patch*`, `confirm*` | `before` and `after` shapes. Portion change rescales nutrition. Refuses on recipe-splits and legacy flat-nutrition items. |
| `create_food` | Add a new food to the catalog | `name*`, `portion*`, `unit*`, `nutrition*`, `brand`, `category`, `barcode`, `notes`, `confirm*` | New food id + normalized fields. Rejects duplicates (case-insensitive name+brand). Caps per-nutriment values at 100000 to keep hallucinated numbers out of the catalog. |

### NutriTrace scoping guarantees

Every tool query prepends `WHERE user_id = ?`. A token cannot read, write, or destroy anything outside its owner's account, even for admins. A static wiring test (`scripts/mcp-wiring.test.js`) fails CI if a future tool omits the scoping clause.

Tokens can hold any combination of scopes independently. Common configurations:

- **Read-only** (`mcp:read`): a "look at my data" token for read-mostly agents.
- **Read + write** (`mcp:read` + `mcp:write`): typical "help me log stuff" agent.
- **Full** (`mcp:read` + `mcp:write` + `mcp:destroy`): power-user token; every destructive call still needs `confirm: true`.

### NutriTrace rate limits

Same per-token bucket as the Federation API: 60 requests per minute by default (`API_RATE_LIMIT_PER_MIN`). Responses include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (unix seconds). 429s carry `Retry-After`.

### Not currently exposed (NutriTrace)

Deliberately not in the current MCP surface:

- `delete_diary_day` (whole-day nuke). Blocked by tombstone-resurrection semantics; single-entry delete covers the common "I logged the wrong day" case safely.
- `log_recipe`. Recipes vary in yield semantics; logging component foods with `log_food` is the safe path today.
- `get_weight_history`, `list_meals`, `get_food_by_id`. Trivial to add; deferred until real demand.
- MCP prompts capability (guided workflows). Planned; no ETA.

## LiftTrace

Eleven tools across three phases.

### Read tools (`mcp:read`)

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `get_workout` | Full detail of one day's workout | `date` | `date`, `logged`, `name`, `completed`, `duration_min`, `exercises[]` (each with `sets[]`: reps, weight, completed, warmup, rpe) |
| `list_recent_workouts` | Summaries of the most recent logged workouts | `limit` | `workouts[]` (date, name, completed, exercise_count, total_volume), `count` |
| `get_records` | Personal records per exercise — max weight, rep count, date, estimated 1RM | `exercise_name` | `records[]`, `count` |
| `get_exercise_progress` | Per-session progress for one exercise over a range | `exercise_name*`, `start`, `end` | `progress[]` (date, maxWeight, totalVolume, sets, avgRpe). Ambiguous name matches return a `candidates[]` disambiguation list instead of guessing. |
| `search_exercises` | Search the exercise catalog by name | `query*`, `limit` | `exercises[]` (exercise_id, name, category, equipment, load_type), `count` |
| `list_programs` | The caller's programs, owned or coach-assigned | none | `programs[]` (program_id, name, duration_weeks, is_active, template_count), `count` |
| `get_active_program` | The active program's current week + every weekly template | none | `active`, `program_id`, `name`, `current_week`, `templates[]` (with each template's exercises) |
| `get_body_stat` | Weight / body-fat / measurements for a day | `date` | `date`, `logged`, `stats` |

### Write tools (`mcp:write`)

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `log_set` | Append one completed set to an exercise on a day | `exercise_id*`, `reps*`, `weight*`, `rpe`, `warmup`, `completed`, `date` | `ok`, `exercise_id`, `exercise_name`, `logged_set`, `sets_on_exercise`. Creates the exercise entry for that day if it isn't there yet; requires a real `exercise_id` from `search_exercises` (does not create new catalog exercises). Goes through the same Option C merge (`mergeExercises`) the app's own save uses, so a concurrent app save landing mid-call can't be clobbered. |
| `log_body_stat` | Set weight / body-fat / measurements on a day | `weight`, `weight_unit` (`kg`\|`lb`), `bodyFat`, `waist`, `hips`, `neck`, `chest`, `biceps`, `thighs`, `calves`, `date` | `ok`, `date`, `logged`, `stats`. Merges into existing values via `mergeStatsObject`, which allowlists the known body-stat keys — a malformed payload can't land stray fields in the stored JSON. |

### Destructive tools (`mcp:destroy` + `confirm: true`)

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `delete_workout` | Permanently delete a day's entire workout | `date*`, `confirm*` | `ok`, `deleted`, `date`. Hard delete, matching `DELETE /api/workout/:date`'s existing semantics — no soft-delete step. |

### LiftTrace scoping guarantees

Every tool query scopes on an ownership column — `user_id` for `workout_log` / `body_stats_log`, `assigned_to` for `program_assignments`, `created_by` for `exercises` / `programs` (with `is_global = 1` rows intentionally shared across every user — that's the point of a global exercise, not a leak). A token cannot read, write, or destroy anything outside its owner's account, even for admins. A static wiring test (`scripts/mcp-wiring.test.js`) fails CI if a future tool omits the scoping clause. MCP tokens always belong to a real user account — see [the MCP setup page](../lifttrace/mcp.md) for why API Tokens only appears on a multi-user instance.

### LiftTrace rate limits

Same per-token bucket model as NutriTrace's: 60 requests per minute by default (`API_RATE_LIMIT_PER_MIN`). Responses include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (unix seconds). 429s carry `Retry-After`.

### Not currently exposed (LiftTrace)

Deliberately not in the current MCP surface:

- `edit_workout` / `edit_set`. `log_set` covers the primary "log what I just did" case; patching an existing set adds ambiguity (which set, by what identity) without a clear demand signal yet.
- `log_workout` (a whole ad-hoc session in one call). `log_set` per-set is simpler and safer to reason about for a first pass; revisit if agents end up chaining many `log_set` calls per session.
- Superset detail, cardio sessions, coach prescriptions. Trivial to add; deferred until real demand.
- MCP prompts capability (guided workflows). Planned; no ETA.

## Related

- [Model Context Protocol setup (NutriTrace)](../nutritrace/mcp.md): turn it on, wire up Claude Desktop
- [Model Context Protocol setup (LiftTrace)](../lifttrace/mcp.md): turn it on, wire up Claude Desktop
- [Trace tool catalog](trace-tools.md): in-app Trace AI tools (separate mechanism)
- [Federation API](../nutritrace/federation-api.md): the older `/api/v1/*` REST surface that NutriTrace's MCP shares its token model with
