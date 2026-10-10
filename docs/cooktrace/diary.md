# Cook Diary and Meal Planner

Both live on the same page (`/diary`). The distinction is a single column on each row: `kind = 'cooked'` for a log entry, `kind = 'planned'` for something scheduled. That means the same UI, the same drag-and-drop, and one-tap promotion from planned to cooked.

## List view

Rows grouped by date, 60 days back and 30 days forward by default. Each row shows the recipe hero, name, meal type, per-cook rating, notes, and a status pill (Cooked / Planned). Planned rows have a **Mark as Cooked** button that flips `kind` and stamps the entry to today (or keeps the planned date if you'd rather).

Add a cook or a plan from the **+** button: pick a recipe, date, and meal type (Breakfast / Lunch / Dinner / Snack, or unset). Notes and a per-cook rating are optional.

## Month calendar

Toggle to Calendar for a month grid with recipes as chips inside each cell. Drag a chip between cells to reschedule. Drag between planned and today's cell to "cook now". The calendar respects the same `meal_type` colouring and shows a small stack when a day has more than three entries (tap to expand).

## Plan-then-cook workflow

The typical week:

1. Open Diary, Calendar view, forward one week.
2. Drop recipes into the days you want them from the recipe picker.
3. Open Shopping and use **Add from Planned Cooks** to put every planned recipe's ingredients on the list, scaled to each cook's servings and added up across recipes. Building it again after changing the plan updates the list rather than doubling it.
4. During the week, tap **Mark as Cooked** as you go.

Planned entries that lapse without being marked cooked stay planned; delete or drag them to a new date if you want to keep the calendar honest.

## Heatmap

The top of the Diary page shows a contribution-graph-style **heatmap** of cook density over the last year. Colours pull from your accent colour with luminance steps by cook count. Click a cell to jump to that day's row in the list. Backed by `GET /api/cook-diary/heatmap`.

## Per-recipe cook history

Every recipe view shows its own cook history as a table (date, meal type, rating, notes). Handy when you want to remember "did I use the small pan or the big pan last time?" and the note you left past-you is right there.

## Meal-plan-to-shopping-list

**Add from Planned Cooks** on the Shopping page (`POST /api/shopping/from-plan`):

- Walks every planned entry in the chosen date range, scaling each recipe to the entry's servings.
- Adds up the same ingredient across recipes and across units of one kind (cups with millilitres, grams with kilograms).
- Skips anything stocked in the pantry (via each ingredient's pantry link), and anything already checked off on the list.
- Keeps, on each list item, the recipes and planned cooks it's for, so building the same range again replaces their share instead of adding it twice, and a cook taken off the plan takes its share out.

`POST /api/shopping/from-recipe/:id` handles a single recipe ("Add to Shopping List" on a recipe card), scaled with `?servings=` when given.

See [Shopping](shopping.md#add-from-planned-cooks) for the details.

## Data model

`cook_diary` carries: `user_id`, `recipe_id` (nullable, in case the recipe is deleted later), `date` (`YYYY-MM-DD`), `kind` (`cooked | planned`), `servings`, `meal_type`, `rating`, `notes`, timestamps. Deletes are soft (`deleted_at`), which keeps `last_cooked_at` roll-ups on the recipe honest during sync.

## Related

- [Recipes](recipes.md)
- [Pantry](pantry.md)
- [Feature tour: Shop from the plan](features.md#shop)
