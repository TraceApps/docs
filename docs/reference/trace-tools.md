# Trace tool catalog

Every tool Trace can call, across all three apps, in one table. Use this when you are writing prompts, debugging a "Trace refused to do X" report, or wondering whether an app can do a thing conversationally.

**About the columns:**

- **Tool** is the exact `name` the model sees. Present in `src/lib/aiChat.js` under `export const TOOLS`.
- **App** is which of the three exposes it. LiftTrace intentionally has no tool schema (see below).
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

**LiftTrace does not expose a Trace tool schema.** Instead it pre-computes a context block (`buildContext()` in `src/components/ai/Trace.svelte`) and injects it into the system prompt. The block includes user profile, active program, recent workouts (last 14 days), body stats, and PR summaries. Trace answers from that snapshot rather than reaching for live tools.

For write actions, LiftTrace uses **Smart Log**, a separate hold-to-record UI (not a Trace tool) that parses natural-language workout entries (`bench 3x5 @ 225, squats 5x5 @ 315`) with the `smartLogWorkout.js` parser, matches exercises against the user's library, and pre-fills a review modal. The user commits by tapping Save.

This is a deliberate design call: workout logging is precise (weight, reps, sets, RPE), and passing it through an LLM tool round-trip adds latency without adding correctness. The parser handles the common shapes directly.

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

CookTrace and NutriTrace both cap the tool-call loop at **5 rounds** per user message. If Trace has not converged on a final text answer within 5 rounds it stops and returns whatever it has. This matches the loop cap set in `_callClaudeWithTools`, `_callOpenAIWithTools`, and `_callGeminiWithTools`.

Tool execution stays **client-side** even when the server-side AI proxy is in use (`AI_ENABLED=1` env-locked mode). The proxy relays messages and returns tool-call requests; the client executes each tool against the user's local database and UI state, then loops. This keeps the API key on the server without giving the server write access to the user's data.

## Working with tools when writing prompts

- **Read tools accept date ranges** (`from` / `to` in `YYYY-MM-DD`). Ask "what did I cook last week?" and Trace will call `get_diary` with the appropriate range.
- **Write tools are single-round.** A follow-up question like "and log two more" needs an explicit prompt; Trace does not batch writes speculatively.
- **`propose_*` on NutriTrace does not log.** If Trace says "I logged X" after calling `propose_food` or `propose_quick_calories`, that is a hallucination; the actual entry only lands after the user taps **Add to Diary** or **Save & Add to Diary** on the review card.
- **Trace can call one tool per round.** If your question implies multiple lookups ("compare last week to this week"), Trace will call the first, receive the result, then call the second in the next round.

## Related

- [Setting up Trace](../trace/setup.md)
- [Cloud providers](../trace/cloud.md)
- [Local LLMs](../trace/local-llm.md)
- [Model list and Gemini retirement remap](../trace/models.md)
