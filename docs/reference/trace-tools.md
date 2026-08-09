# Trace tool catalog

## What tools are and why they matter

A **tool** in this context is a function the AI model can decide to call on its own, mid-conversation. When you ask Trace "what can I make from what's in my pantry?", the model doesn't guess. It calls the `get_pantry` tool, gets your actual stock back, then answers using real data. Same for "log a cook of Thursday's pasta", "how many calories yesterday?", or "add tomatoes to shopping": the model picks the right tool, fires it, reads the result, and writes the reply grounded in what it saw.

This is called *function calling* in OpenAI's terminology, *tool use* in Anthropic's. Same idea. The practical effect: Trace answers are grounded in your live app state instead of made up from the model's training corpus. It can also mutate state, log a cook, add a pantry row, plan a meal, so the assistant becomes a hands-on second UI rather than a chatbot.

All three TraceApps (CookTrace, LiftTrace, NutriTrace) now expose tool schemas to Trace. Per-app catalogs follow.

## About this page

Every tool Trace can call, across all three apps, in one table. Use this when you are writing prompts, debugging a "Trace refused to do X" report, or wondering whether an app can do a thing conversationally.

**About the columns:**

- **Tool** is the exact `name` the model sees. Registered under `export const TOOLS` in each app's `src/lib/aiTools.js` (or `src/lib/aiChat.js` on older apps).
- **App** is which of the three exposes it.
- **Purpose** is the one-line description the model reads at call time.
- **Args** lists parameter names; `*` marks required.
- **Returns** describes the shape Trace gets back, so you can predict how it will phrase the reply.

## CookTrace

Nineteen tools; the biggest surface of the three. Read tools return recipe, pantry, diary, shopping-list, and cookbook state. Write tools log a cook, plan a cook, add pantry rows, add shopping items, import a recipe from a URL, and create a recipe from scratch.

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `get_recipes` | Recipe library with pantry-match ratio | `query`, `category`, `favorite` | List of recipes (id, name, servings, times, rating, pantry_match) |
| `list_recipe_categories` | Categories the user has defined | none | Name, slug, color, recipe count per category |
| `get_recipe` | One recipe, full detail | `id*` | Grouped ingredients, steps, tools, tags, per-serving nutrition |
| `get_pantry` | Pantry inventory | `in_stock_only`, `query`, `category` | Rows with brand, in_stock, quantity, unit, expiry, nutrition |
| `list_pantry_categories` | Pantry categories | none | Name, slug, icon per category |
| `find_recipes_from_pantry` | "What can I cook tonight?" | `min_ratio` | Recipes ranked by have/need ratio |
| `get_diary` | Past and planned cooks | `from*`, `to*` | Date, kind (`cooked` or `planned`), recipe, meal_type, rating |
| `get_shopping_list` | Current shopping list | none | Name, quantity, unit, aisle, checked, recipe link |
| `get_cookbooks` | Cookbook collections | none | Id, name, recipe_count, smart_filter |
| `get_cookbook` | One cookbook with its recipes | `id*` | Cookbook plus resolved recipe list |
| `log_cook` | "I cooked this" | `recipe_id*`, `date`, `notes`, `meal_type`, `rating` | Updates last_cooked_at + cook_count |
| `plan_cook` | "Plan tacos for Friday" | `recipe_id*`, `date*`, `notes`, `meal_type` | Creates a `planned` diary entry |
| `add_to_shopping` | Add one or more items | `items*` (array of `{name, quantity?, unit?, aisle?}`) | Auto-links to pantry rows by name |
| `add_to_pantry` | New pantry item or update by name | `name*`, `in_stock`, `quantity`, `unit`, `brand`, `notes` | Pantry row id |
| `set_pantry_density` | Volume-to-weight conversion | `pantry_id*`, `g_per_cup*` | Enables cross-family nutrition calc |
| `set_pantry_stock` | "I'm out of butter" | `pantry_id*`, `in_stock*` | Boolean flip |
| `add_to_cookbook` | Add recipes to a regular cookbook | `cookbook_id*`, `recipe_ids*` | Updated recipe list |
| `import_recipe_from_url` | Scrape schema.org/Recipe | `url*`, `add_to_pantry`, `apply_tags` | New recipe id in the library |
| `create_recipe` | Dictated or photo-imported recipe | `name*`, `ingredients*`, `steps*`, plus optional metadata | New recipe id |

## LiftTrace

Eighteen tools spanning read and write across workouts, exercises, programs, PRs, body stats, and coaching. Read tools surface diary, exercise catalog, active + saved programs, personal records, body-stat time series, weekly rollups, and coach prescriptions. Write tools log a full workout, drop a single exercise onto today's diary, append a set to a workout in flight, log a body-stat measurement, load a template into a diary day, switch the active program, or (for coaches only) prescribe a workout to a trainee.

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `get_workouts` | Recent workouts with completed sets, per-exercise volume, and top set | `date_from`, `date_to`, `exercise_name` | Shaped workout rows; default window is last 30 days |
| `get_workout` | One workout in full detail by id, including coach feedback | `id*` | Every set, notes, coach feedback, program context |
| `get_exercises` | Search the exercise catalog (wger, free-exercise-db, exercisedb variants, custom) | `query`, `muscle`, `equipment`, `source`, `limit` | Compact rows with id, name, muscles, equipment, source |
| `get_exercise` | One exercise's full detail plus similar exercises | `id`, `name` (id wins if both) | Description, instructions, muscles, equipment, up to 5 similar |
| `get_programs` | User's program library with active flag | none | Id, name, goal, template count, weeks, is_active |
| `get_program` | One program with every template laid out | `id*` | Templates with target sets/reps/load/RPE/tempo/rest, per-week overrides |
| `get_active_program` | The currently active program in full | none | Same shape as `get_program`; `{ active: false }` if none |
| `get_prs` | Personal records with e1RM | `exercise_name`, `date_from`, `date_to`, `limit` | Top weight, top reps at that weight, estimated 1RM per exercise |
| `get_body_stats` | Body stats over time (weight, body_fat, any tracked measurement) | `stat`, `date_from`, `date_to` | Time series plus first/last/min/max/change summary when a single stat is queried |
| `get_stats_overview` | Snapshot over a window: workouts done, streaks, weekly volume + frequency, muscle balance, top PRs | `range` (`7d`\|`30d`\|`90d`\|`1y`\|`all`) | Rollup object; default range `30d` |
| `get_coach_prescription` | Coach-prescribed workout for a given date if the user has a trainer | `date` (default today) | `{ prescribed: false }` or the prescription record |
| `log_workout` | Commit a full workout to the diary for a date | `date*`, `exercises*`, `name`, `duration_min` | Workout id, exercises_logged, sets_logged, total_volume |
| `add_exercise_to_diary` | Quick-add one exercise to a diary day with no sets yet | `exercise_name*`, `date` | Workout id, exercise_added_id, position |
| `log_set` | Append a single set to an exercise already in a workout | `workout_id*`, `weight*`, `reps*`, `exercise_id` or `exercise_name`, `rpe`, `warmup`, `completed` | Set id, position, workout total volume |
| `log_body_stat` | Record a body-stat measurement for a date; merges with existing | `stat*`, `value*`, `unit`, `date`, `note` | Stats row id, stat, value, unit, date |
| `start_workout_from_template` | Load a template into a diary day | `template_id` or `template_name`, `date` | Workout id, exercises_loaded, from_template |
| `set_active_program` | Switch the user's active program | `program_id` or `program_name` | New active_program_id, previous_active_program_id, name |
| `add_coach_prescription` | COACH ONLY. Prescribe a workout template to a trainee | `trainee_id*`, `template_id*`, `target_date*`, `notes` | prescription_id, trainee_id, template_id, target_date |

LiftTrace also keeps **Smart Log**, a hold-to-record UI on the Trace FAB (not a Trace tool) that parses natural-language workout entries (`bench 3x5 @ 225, squats 5x5 @ 315`) client-side with the `smartLogWorkout.js` parser, matches exercises against the user's library, and pre-fills a review modal. Faster than a tool round-trip for the common case; the user commits by tapping Save.

## NutriTrace

Sixteen tools covering diary reads, wellness reads, and structured writes. The `propose_*` tools are photo-review paths; they display a card the user must confirm before anything writes.

| Tool | Purpose | Args | Returns |
|------|---------|------|---------|
| `get_wellness_data` | Wearable metrics (steps, sleep, HR, HRV, readiness, VO2) | `from*`, `to*` | Daily rows across enabled sources |
| `get_body_composition` | Withings scale data | `from*`, `to*` | Weight, body fat %, muscle mass, visceral fat, ECG, segmental |
| `get_diary` | One date's diary | `date*` | Meals, items, nutrition, water, day notes, activities |
| `get_meals` | Saved Meals and Recipes library | `query` | Items, totals, notes |
| `get_workouts` | Wearable-synced workouts | `from*`, `to*` | Name, duration, distance, HR, GPS availability |
| `get_goals` | Nutrition and wellness targets | none | Calorie, macro, nutrient targets |
| `add_activity_entry` | Log a manual exercise | `name*`, `kcal*`, `date`, `duration_min`, `distance`, `source`, `met`, `is_template` | Diary Activity row |
| `get_diary_averages` | Averages over N days | `days*` | Averages, days_logged/period_days, weight change |
| `get_logging_streak` | Consecutive-day streak | none | `streak_days`, `streak_start`, `streak_end`, `today_logged` |
| `get_fasting_history` | Intermittent fasting history | `days` | Last N fasts plus summary stats |
| `get_adaptive_tdee` | Learned TDEE from 35-day regression | none | `ready`, `tdee`, `trendKgPerWeek`, `confidence`, `daysAvailable` |
| `log_food` | Log a real food with full nutrition | `food*`, `meal*`, `portion`, `unit`, `quantity`, `date` | Logged / candidates / no_match / error |
| `log_quick_calories` | Kcal-only diary row (Fitbit / MFP style) | `meal*`, `kcal*`, `name`, `protein_g`, `carbs_g`, `fat_g`, `date` | Diary row id |
| `propose_quick_calories` | Photo-based Quick Calories, user must confirm | `name*`, `nutrition*`, `meal`, `date`, `serving_grams`, `serving_size` | Review card shown; nothing logged yet |
| `propose_food` | Photo-based reusable food, user must confirm | `name*`, `nutrition*`, `brand`, `portion`, `unit`, `meal_hint`, `notes` | Review card shown; nothing saved yet |
| `get_activity_log` | Manually-logged activities | `from*`, `to*` | Name, kcal, duration, distance, source per entry |

## Tool-use loop

All three apps cap the tool-call loop at **5 rounds** per user message. If Trace has not converged on a final text answer within 5 rounds it stops and returns whatever it has. This matches the loop cap set in `_callClaudeWithTools`, `_callOpenAIWithTools`, and `_callGeminiWithTools`.

Tool execution stays **client-side** even when the server-side AI proxy is in use (`AI_ENABLED=1` env-locked mode). The proxy relays messages and returns tool-call requests; the client executes each tool against the user's local database and UI state, then loops. This keeps the API key on the server without giving the server write access to the user's data.

## Working with tools when writing prompts

- **Read tools accept date ranges** (`from` / `to` in `YYYY-MM-DD`). Ask "what did I cook last week?" and Trace will call `get_diary` with the appropriate range.
- **Write tools are single-round.** A follow-up question like "and log two more" needs an explicit prompt; Trace does not batch writes speculatively.
- **`propose_*` on NutriTrace does not log.** If Trace says "I logged X" after calling `propose_food` or `propose_quick_calories`, that is a hallucination; the actual entry only lands after the user taps **Add to Diary** or **Save & Add to Diary** on the review card.
- **Trace can call one tool per round.** If your question implies multiple lookups ("compare last week to this week"), Trace will call the first, receive the result, then call the second in the next round.

## Not to be confused with MCP tools

The tools on this page are what NutriTrace's built-in **Trace AI** (the in-app assistant) can call. NutriTrace also exposes a completely separate set of tools via the **Model Context Protocol** for *external* AI clients like Claude Desktop, Cursor, and Codex — those talk to `/api/mcp` over HTTP and don't share code with Trace. Different mechanism, different tool schemas, different auth model (bearer scopes instead of in-app AI settings).

See the dedicated [MCP tool catalog](mcp-tools.md) for the external-agent side.

## Related

- [Setting up Trace](../trace/setup.md)
- [Cloud providers](../trace/cloud.md)
- [Local LLMs](../trace/local-llm.md)
- [Model list and Gemini retirement remap](../trace/models.md)
- [MCP tool catalog](mcp-tools.md) — external agents (Claude Desktop, Cursor, Codex)
