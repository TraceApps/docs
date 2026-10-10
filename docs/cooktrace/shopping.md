# Shopping list

The Shopping page (`/shopping`) is a quick-add list with per-user grouping, aisle awareness, drag-to-reorder, and one-tap bulk clear. Rows can be typed in by hand, pushed from a single recipe, or fanned out from a whole week of planned cooks.

## Grouping

Under Settings > Cooking > Shopping List > Default Grouping, pick one of:

- **By Aisle** (default): every row groups under its aisle heading (Produce, Dairy, Bakery, and so on). Aisle comes from the row's linked pantry item via its category's `default_aisle`. Untagged rows fall under a trailing "Other" group.
- **By Recipe**: rows group under whichever recipe pushed them. Manual rows (no `recipe_id`) sit in an "Other" section at the end.
- **Flat**: no groups. A single sorted list.

Drag any row inside its group to reorder. Cross-group drag (drop a row into a different aisle) works too. The order persists via `POST /api/shopping/reorder`, so a hand-arranged aisle stays put across page loads.

### Combined items

In **By Aisle** and **Flat**, rows with the same name and unit show as a single row. Add Pancakes and Bread and you get one "Flour 3 cup" row with a pill for each recipe, instead of two Flour rows. The amounts are added up; if any of them has no amount, the combined row shows none rather than a wrong total. Different units stay separate, so "2 cup flour" and "200 g flour" are two rows.

Checking, removing, dragging, or changing the aisle of a combined row applies to every row behind it. In **Edit**, changing only the name or unit updates them all and keeps each recipe's own amount; changing the amount folds them into one row with the new amount.

**By Recipe** doesn't combine anything, so each recipe still shows its own rows, and the recipe block's trash icon removes only that recipe's share. The list is stored one row per recipe either way; combining only changes how it's shown.

## Checked-item behavior

Tap the checkbox and the row strikes through and drops to the bottom of the list; the DB flips `checked = 1` and the sort clause slots checked rows last. Under Settings > Cooking > Shopping List > Checked Items you can switch this to **Hide** if you'd rather not see checked rows at all.

Neither mode deletes anything. Rows stay put until you hit **Clear Checked** on the toolbar, which soft-deletes every checked row in one call (`DELETE /api/shopping/checked`). That's the "wipe the trolley" moment when you're home.

### Restock the pantry

**Clear Checked** asks before clearing, and lists the pantry items you've just bought that are still marked out of stock, all ticked. Untick anything you didn't actually get, then **Clear**: the list is cleared and the ticked items go back in stock, so there's no second trip through the Pantry tab. The toast that follows has an **Undo** that puts those pantry items back out of stock.

An item is only offered when it's clearly the same thing: the shopping row is linked to a pantry item, or its name matches exactly one pantry item (ignoring case). If there's no match, or more than one, it isn't listed and nothing in the pantry changes. A [generic parent](pantry.md#variants) such as Milk has no stock of its own, so it shows a dropdown for which variant you bought, starting on its designated variant if one is set. A generic already covered by an in-stock variant, like any item already in stock, isn't listed.

Restocking marks the item in stock and clears a quantity of 0. Expiry dates and other quantities are left as they are.

## Add from a recipe

Long-press a recipe card and choose **Add to Shopping List**, or use **Add from Recipe** in the Shopping page's add menu. Every ingredient lands on your list, stamped with `recipe_id` so the By-Recipe view can badge the block and so the small trash icon on that block can wipe all of that recipe's rows in one shot (`DELETE /api/shopping/by-recipe/:id`).

The picker's **Only add items I don't have in pantry** toggle (on by default) uses each ingredient's pantry link plus the item's `in_stock` value to skip anything you already have. Turn it off to force the full ingredient list onto the shopping list regardless of stock. See [Pantry](pantry.md) for how stock tracking works.

## Add from Planned Cooks

In the Shopping page's add menu, **Add from Planned Cooks** asks for a date range (today plus 7 days to begin with) and puts every planned cook in it on the list (`POST /api/shopping/from-plan`):

- Each recipe is scaled to the servings planned for that cook. A recipe written for 2 and planned for 4 adds twice its amounts. A planned cook with no servings, or a recipe with none, uses the recipe as written.
- The same ingredient from several recipes becomes one item. Names match regardless of case and plurals ("Tomatoes" and "tomato"), and amounts add up across units of one kind: 1 cup and 120 ml of milk become one amount, shown in the unit that reads best. Amounts that can't be added up (grams and cups, or cloves and pieces) stay as separate items.
- Fractions, mixed numbers and ranges count: "1/2", "1 1/2", "½", and "4-5" (the larger number, so there's enough). A line without an amount ("salt, to taste") adds the item with no amount, without wiping out the amounts from other recipes.
- Every item remembers which recipes and planned cooks it's for.
- Building the list for the same range again updates it instead of adding everything twice. A cook taken off the plan, or moved out of the range, takes its share out, and an item only that cook needed leaves the list. An amount you typed into an item yourself stays. Items you've already checked off count as bought and aren't added back.
- An item that already exists on the list, added by hand or from a recipe, gets the plan's amounts added to it rather than a second row.
- **Only add items I don't have in pantry** (on to begin with) leaves out ingredients whose linked pantry item is in stock.

Adding a single recipe works the same way, except that each time adds again: you might cook it twice.

See [Cook Diary and Meal Planner](diary.md) for how planned cooks land in the calendar.

## Manual rows

The quick-add row at the top of the page takes a name, optional quantity, optional unit, and optional aisle. Quantities take fractions as well as decimals ("1/2", "1 1/2"), and the list shows them the way a cook writes them: 3/8 cup rather than 0.375 cup for cups and spoons, decimals for grams and milliliters. Things you count round up to whole ones when a plan or recipe adds them (8 tomatoes rather than 7.5). Names get title-cased on save so the list reads consistently regardless of source (Mealie's canonical `"fresh lemon juice"` becomes `Fresh Lemon Juice`; small connective words like "and" or "of" stay lowercase per Chicago rules).

## Shared list?

The list is per-user. There's no built-in "share the shopping list with my flatmate" flow in v1.0. If both users are in a [Kitchen](kitchens.md), the practical pattern is: one person runs **Add from Planned Cooks** on the shared plan, everyone else shops from their own screen, and only the person who checked something has that row checked.

## From NoteTrace

If you keep lists in [NoteTrace](../notetrace/index.md), a checklist there can send its open items straight to this shopping list, and NoteTrace can show this list, grouped by aisle, and check items off. Items it sends are title-cased, take the aisle of a pantry item with the same name, and aren't added twice when they're already on the list.

It needs no server setting here. In CookTrace **Settings, API Tokens**, create a token with just the **shopping** scope (it reaches nothing but your shopping list) and paste it into NoteTrace. See [Send to CookTrace](../notetrace/cooktrace.md).

## Related

- [Recipes](recipes.md)
- [Pantry](pantry.md)
- [Cook Diary and Meal Planner](diary.md)
