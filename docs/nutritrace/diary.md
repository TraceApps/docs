# Diary & meal logging

The Diary is the default landing page. One day per view, one card per meal, a nutrition bar at the top, and a body-stats / water / activity / IF strip depending on which sections you have enabled.

## Meal slots

The default slots are **Breakfast, Lunch, Dinner, Snacks**. Rename them, add more, or reorder in **Settings → Diary → Meal names**. The list is a plain array; the Diary picks it up on the next render and Smart Log's meal-slot detection uses the same list, so a name like "Post-workout" doubles as both a diary bucket and a keyword Trace can route matches to.

Storage lives per-day in the `diary` table (`server/db.js:57-68`). Empty slots are hidden by default; toggle `Show empty meals` if you prefer the full grid.

## Add a food, meal, or recipe

Tap **+ Add** on any meal card. The picker opens with search across your Foods, Meals, and Recipes. The **source filter chips** across the top scope the results: Local (your saved foods), OFF (live Open Food Facts), USDA (live FoodData Central lookup), Mealie (your Mealie recipes if configured), and From Others (foods other users on this instance have shared with you). Long-press a chip to combine sources into a multi-source view; each result row carries a source badge and, for OFF, an origin-country flag and a completeness dot (see [Food data quality signals](../reference/food-data-quality.md)).

Pick a food. The quantity sheet opens with a **live nutrition preview**: calories, protein, carbs, and fat update as you change the amount, unit, or number of servings. Food notes ("1 serving = 150 g cooked", "half a bag") surface at the top of the sheet. Meals and recipes carry the same preview and expand to their component foods on save.

## Long-press and right-click

Long-press a diary item on mobile, right-click on desktop, for the item action sheet: **Edit**, **Move to another meal**, **Copy to another meal**, **Copy to another date**, **Delete**. The same interaction on the meal card header (title area) opens the meal `⋮` menu.

## Per-meal `⋮` menu

The meal-level `⋮` (top-right of each meal card) covers meal-wide actions:

- **Copy meal to another meal**: same day, different slot.
- **Move meal to another meal**: same, but the source empties.
- **Copy meal to another date**: pick any date; the items land in the same slot on that date.
- **Save meal to library**: creates a saved Meal in your Foods library from the current slot contents, so you can quick-add the whole thing tomorrow.
- **Clear meal**: wipes the slot.

## Split Recipe

Log a recipe (say "Chili con carne, 1 serving") and the diary shows one row with the recipe totals. Long-press the row and pick **Split Recipe**. The row expands in place into its component ingredients, each scaled to the serving you logged, each individually editable. Delete one, change a quantity on another, and the meal totals recompute correctly. The recipe row itself is gone after the split; there is nothing to un-split, so save any tweaks before you commit.

Useful when the recipe was close but you actually used more oil than the recipe calls for, or you left out an ingredient.

## Nutrition bar

The bar at the top of the Diary shows the day's calorie total, remaining calories against your goal, and a ring or bar per macro. Toggles in **Settings → Diary**:

- `Totals mode`: consumed vs remaining.
- `Macro legend`: off / percent / grams / both.
- `Show all nutrients`: expand micros in the summary sheet.
- `Show brand / timestamp / thumbnail` on each row.

Tap the bar to open the **Nutrition Summary sheet**: every tracked nutrient in your configured order (Settings → Nutrients), plus a per-meal breakdown. Custom nutrients defined in Settings show up here alongside the built-ins.

## Body stats

A collapsible section on the Diary holds body stats (weight, waist, chest, custom entries). Reorder or hide fields under **Settings → Body Stats**; values are stored on the day's `diary.body_stats` blob. Connected-scale readings take priority over manual entries when Adaptive TDEE consumes them, but manual entries fill any gaps.

## Water

Water is its own card. Container library, unit (ml or fl oz), and daily goal live under **Settings → Water**. Tap a container to add a serving; long-press for edit. Water writes go straight to SQLite on Android (unlike diary items, which are debounced), so a container tap is immediately persisted. See the persistence caveat under **Known caveats** below.

Smart Log recognizes water phrasing ("500 ml water", "a big glass of water") and routes those matches to the water log, not the food diary.

## Activity

Enable **Settings → Diary → Activity section** to add a manual workout row directly from the Diary: name, calories, duration, distance, MET. Templates (rows marked `is_template=1`) turn repeat workouts into one tap. AI kcal estimate is a separate toggle; when on, Trace fills the calorie field from your body profile and a plain-language workout description.

`manualActivityPolicy` decides how a manual entry interacts with a wearable entry on the same day:

- `wearable_wins` (default): wearable data is authoritative; manual is ignored for goal math.
- `manual_wins`: manual takes precedence.
- `additive`: both are summed.

Deep dive: [Activity & Intermittent Fasting](activity-if.md).

## Intermittent Fasting

Toggle `fastingEnabled` and the IF widget appears above the meal cards. Presets are `14:10`, `16:8`, `18:6`, `20:4`, `OMAD`, and `custom`. Start a fast; a live elapsed timer runs until you tap **End** or the goal-reached notification fires. Custom presets and scheduled auto-starts live under **Settings → Diary → Fasting**. Endpoints: `GET /api/fasts`, `GET /api/fasts/active`, `POST /api/fasts/start`, `POST /api/fasts/:id/end`, `PATCH/DELETE /api/fasts/:id`.

## Per-day notes

A free-text notes field per date, toggled by `diaryShowNotes`. Handy for context that shifts the numbers ("felt bloated after lunch", "post-workout"). An indicator dot renders on dates that have a note.

## Known caveats

!!! warning "Android diary persistence gap"
    Diary writes on Android are debounced to SQLite; water writes are immediate. If the app is killed before the debounce fires (long stretches with the phone screen off, an aggressive battery saver, or an OS OOM), the affected diary items are lost on next launch. Water is unaffected. Open incident tracked upstream; workaround is to switch dates or background the app briefly after a large log to force a flush.

## Related

- [Foods, meals & recipes](foods.md)
- [Goals & Adaptive TDEE](goals.md)
- [Trace in NutriTrace + Smart Log](trace.md)
- [Activity & Intermittent Fasting](activity-if.md)
