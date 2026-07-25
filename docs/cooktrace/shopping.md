# Shopping list

The Shopping page (`/shopping`) is a quick-add list with per-user grouping, aisle awareness, drag-to-reorder, and one-tap bulk clear. Rows can be typed in by hand, pushed from a single recipe, or fanned out from a whole week of planned cooks.

## Grouping

Under Settings > Cooking > Shopping List > Default Grouping, pick one of:

- **By Aisle** (default): every row groups under its aisle heading (Produce, Dairy, Bakery, and so on). Aisle comes from the row's linked pantry item via its category's `default_aisle`. Untagged rows fall under a trailing "Other" group.
- **By Recipe**: rows group under whichever recipe pushed them. Manual rows (no `recipe_id`) sit in an "Other" section at the end.
- **Flat**: no groups. A single sorted list.

Drag any row inside its group to reorder. Cross-group drag (drop a row into a different aisle) works too. The order persists via `POST /api/shopping/reorder`, so a hand-arranged aisle stays put across page loads.

## Checked-item behaviour

Tap the checkbox and the row strikes through and drops to the bottom of the list; the DB flips `checked = 1` and the sort clause slots checked rows last. Under Settings > Cooking > Shopping List > Checked Items you can switch this to **Hide** if you'd rather not see checked rows at all.

Neither mode deletes anything. Rows stay put until you hit **Clear Checked** on the toolbar, which soft-deletes every checked row in one call (`DELETE /api/shopping/checked`). That's the "wipe the trolley" moment when you're home.

## Add from a recipe

Open any recipe and tap **Add to Shopping**. Every ingredient lands on your list, stamped with `recipe_id` so the By-Recipe view can badge the block and so the small trash icon on that block can wipe all of that recipe's rows in one shot (`DELETE /api/shopping/by-recipe/:id`).

The picker's **Only add items I don't have in pantry** toggle (on by default) uses each ingredient's pantry link plus the item's `in_stock` value to skip anything you already have. Turn it off to force the full ingredient list onto the shopping list regardless of stock. See [Pantry](pantry.md) for how stock tracking works.

## Shop This Plan

Open the Diary page, hit **Shop This Plan**, pick a date range (defaults to today plus 7 days), and `POST /api/shopping/from-plan` walks every planned cook in that window and merges their ingredients before writing:

- Same name plus same unit collapses into a single row.
- Numeric quantities sum when every contributor has one. If any contributor is qty-less ("flour", no number), the merged row goes qty-less rather than lie about totals.
- The pantry-stocked skip applies the same way, controlled by the same "Only add items I don't have in pantry" toggle.

See [Cook Diary and Meal Planner](diary.md) for how planned cooks land in the calendar.

## Manual rows

The quick-add row at the top of the page takes a name, optional quantity, optional unit, and optional aisle. Names get title-cased on save so the list reads consistently regardless of source (Mealie's canonical `"fresh lemon juice"` becomes `Fresh Lemon Juice`; small connective words like "and" or "of" stay lowercase per Chicago rules).

## Shared list?

The list is per-user. There's no built-in "share the shopping list with my flatmate" flow in v1.0. If both users are in a [Kitchen](kitchens.md), the practical pattern is: one person runs Shop This Plan on the shared plan, everyone else shops from their own screen, and only the person who checked something has that row checked.

## Related

- [Recipes](recipes.md)
- [Pantry](pantry.md)
- [Cook Diary and Meal Planner](diary.md)
