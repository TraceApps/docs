# Diary & meal logging

The Diary is the default start page. One day per view, one card per meal, a nutrition bar at the top, and a body-stats / water / activity / IF strip depending on which sections you have enabled.

## Meal slots

The default slots are **Breakfast, Lunch, Dinner, Snacks**. Rename them, add more, or reorder in **Settings → Diary → Meal names**. The list is a plain array; the Diary picks it up on the next render and Smart Log's meal-slot detection uses the same list, so a name like "Post-workout" doubles as both a diary bucket and a keyword Trace can route matches to.

Storage lives per-day in the `diary` table (`server/db.js:57-68`). Empty slots are hidden by default; toggle `Show empty meals` if you prefer the full grid.

## Add a food, meal, or recipe

Tap **+ Add** on any meal card. The picker opens with search across your Foods, Meals, and Recipes. The **source filter chips** across the top scope the results: Local (your saved foods), OFF (live Open Food Facts), USDA (live FoodData Central lookup), Mealie (your Mealie recipes if configured), and From Others (foods other users on this instance have shared with you). Long-press a chip to combine sources into a multi-source view; each result row carries a source badge and, for OFF, an origin-country flag and a completeness dot (see [Food data quality signals](../reference/food-data-quality.md)).

Above your foods, **Yesterday's Meals** lists the meals you logged the day before the day you're adding to, one per meal slot; tap one to add all its items to the meal you're adding to. Adding to another day of the diary (tomorrow, when you meal prep) offers the day before that one, and the section is named after it: **From Today**, **From Sunday** (within a week either side), or its date in your date format. Smart Log's "same as yesterday" counts back the same way. Turn the section off in Settings > Foods.

Pick a food. The quantity sheet opens with a **live nutrition preview**: calories, protein, carbs, and fat update as you change the amount, unit, or number of servings. Food notes ("1 serving = 150 g cooked", "half a bag") surface at the top of the sheet. Meals and recipes carry the same preview and expand to their component foods on save.

## Long-press and right-click

Long-press a diary item on mobile, right-click on desktop, for the item action sheet: **Edit**, **Replace**, **Move to Meal**, **Copy**, **Select Multiple**, **Split Recipe** (recipes only) and **Delete**. The same interaction on the meal card header (title area) opens the meal `⋮` menu.

- **Move to Meal** opens with the day you're looking at already picked, so moving to another meal of the same day is one tap on the meal. Pick another day first (see [Picking a day](#picking-a-day)) to move the item there instead.
- **Copy** opens the Copy sheet: a day, a meal, and **Repeat** (below).

## Select Multiple

**Select Multiple** (in the item action sheet) puts check circles on the day's items; tap items in any meal to add or remove them. The top bar shows how many are selected, with four buttons:

- **Cancel Selection** (✕) leaves Select Multiple.
- **Move Selected** moves them all to one meal, on this day or another.
- **Copy Selected** opens the Copy sheet for them. Its meal starts on **Keep Each Item's Meal**: breakfast items go to that day's breakfast, dinner items to its dinner, which is what you want when you plan the same meals ahead. Pick a meal to put them all in that one instead.
- **Delete Selected** asks first, then removes them.

After a move or copy you leave Select Multiple, and a message says where the items went ("Moved 2 items to yesterday's Dinner", "Copied 3 items to 4 days, Thu, Oct 29 to Sun, Nov 1"). Going to another day also leaves it, so a selection never applies to a different day's items.

## Copy and Repeat

The Copy sheet is the same for one item, a whole meal (`⋮` → **Copy**) and a selection. Pick the day and the meal, then use **Repeat** to also copy to the following days: **−** and **+** step from **Off** up to **Next 14 Days**, and the line under it says which days that covers. A copy is a new entry on each day, exactly as if you had copied it there by hand; nothing on those days is replaced or merged.

If some days can't be saved, the others keep their copies and the message says how many days worked and names the ones that didn't, so you can copy those again. A copy made offline waits with your other changes and goes up when the connection returns (see [Offline](features.md#offline)); in the Android app without a server it saves on the phone.

## Picking a day

Everywhere you choose a day for diary items (the Copy sheet, **Move to Meal**, **Move Selected**, and **Add to Diary** on the Foods page) has the same picker: **Yesterday**, **Today** and **Tomorrow** buttons, and the date field with its calendar for any other day. The button for the chosen day is lit; pick a date in the calendar and the matching button lights, or none.

Moving items to another day saves them on that day first and only then takes them off this one. If the first save fails, nothing is removed. If taking them off fails, they are taken back off the other day; if even that can't be done, you're told they are on both days, so nothing is ever lost.

## App icon shortcuts {#app-icon-shortcuts}

Long-press the NutriTrace icon for four shortcuts, all for today:

| Shortcut | Opens |
|---|---|
| **Scan Barcode** | The barcode scanner, on the add screen the meal card's **+** opens. A scan goes on as it does there: the quantity sheet (unless you turned it off in Settings), or the food editor for a code that isn't in your foods or Open Food Facts. |
| **Log Food** | The add screen the meal card's **+** opens, on the tab you used last (Foods, Meals or Recipes). |
| **Log Water** | Today's water card. |
| **Log Weight** | Today's body stats sheet, with the cursor in Weight. |

Scan Barcode and Log Food start on the meal for the time of day. A bar at the top of the add screen says where the food will go ("Adding to today's Lunch"); tap **Change** to pick another meal, or another day with the [day picker](#picking-a-day). After adding, you're on the Diary for that day. The **+** on a meal card works as before: it adds to that meal, with no bar.

The meal for the time of day:

- With **meal reminders** on (Settings → Notifications), each meal's reminder time.
- Otherwise from the meal's name: Breakfast 8:00, a morning snack or brunch 10:30, Lunch 12:30, a snack 15:30, Dinner 18:30, an evening snack 21:00. Common names in French, German, Spanish, Italian, Portuguese and Dutch count too.
- When no name is recognized (Meal 1, Meal 2, ...), the meals in their order, spread from 8:00 to 19:00.

A meal's turn starts an hour before its time and lasts until the next one's. Before the first turn of the day it's the first meal, and late at night (midnight to 4:00) the day's last meal. If the guess is wrong, **Change** fixes it before you add anything.

If water is hidden from the Diary (**Show In Diary** off in Settings → Water), **Log Water** opens that setting and says so, instead of a card you've hidden. Signed out, a shortcut opens the sign-in screen and carries on to its page once you sign in (with a password, fingerprint or single sign-on). They work offline and in the Android app without a server, like the rest of the Diary.

Shortcuts show on Android launchers that support app shortcuts (Android 7.1 and later; most do, including Pixel Launcher, One UI and Nova), and on the installed web app where the browser supports them, such as Chrome and Edge. They aren't on iOS or in Safari. On Android, drag one to the home screen to keep it there.

## Per-meal `⋮` menu

The meal-level `⋮` (top-right of each meal card) covers meal-wide actions:

- **Move Items to…**: every item to another meal of the same day.
- **Copy**: the Copy sheet, for the whole meal. Pick the day and the meal; **Repeat** copies it to the next days too.
- **Save as Meal…**: creates a saved Meal in your Foods library from the current slot contents, so you can quick-add the whole thing tomorrow.
- **Clear All Items**: wipes the slot.

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

## Day completion (opt-in)

Off by default. Turn on under **Settings → Diary → Show Day Completion** to activate every completion surface in one flip. When off, the diary looks exactly as it did before this feature existed. When on:

- **Date bar** grows a check toggle. Tap to close the current day; the icon fills green and a matching check appears next to the date label.
- **Week strip** shows a small green check next to the day number on marked days.
- **Date picker** shows a green corner dot on the calendar cell for marked days.
- **Statistics** (right rail): "Marked complete: N of M days" with a percentage, over the currently viewed range. A row of tiny green dots below the x-axis marks which days in the range were closed.
- **Weekly summary** push and email both include a "Marked complete: X of 7 days" line when at least one day was marked in the window.
- **Bedtime notification** (Android) carries a **Close today** quick action that marks the day complete without opening the app. Skipped when today is already closed so the action isn't offered redundantly.
- **Per-meal checks**: each meal card gets a small tap-to-toggle check next to the meal name. Green when marked, muted when not.
- **Auto-mark day** when every meal slot has been marked complete. Saves a second tap when you're checking off meals one by one.
- **Empty-slot confirm** at day-close: if any meal slot is empty AND not explicitly marked as skipped, a dialog names them ("These meals have no items and haven't been marked as skipped: Breakfast, Snacks") and confirming records the intentional skip so tomorrow's equivalent pattern doesn't re-prompt. Fasters skipping breakfast every day mark it complete once, and it stays cleared.

Sync across devices uses an offline-safe rule: a stale device pushing a `null` day-completion cannot clear a mark another device set, and the per-meal set union-merges across devices so a slot marked on either device stays marked. Explicit unmarking still works from the toggle.

First time you mark a day complete without meal reminders enabled, a one-time tip points at **Settings → Notifications** so you can catch a missing snack earlier next time instead of after end of day.

Storage: `completed_at TEXT` and `completed_meals TEXT` (JSON array of slot indexes) columns on the `diary` table. Both are covered by the full backup export/import and by cross-device sync.

## Known caveats

!!! warning "Android diary persistence gap"
    Diary writes on Android are debounced to SQLite; water writes are immediate. If the app is killed before the debounce fires (long stretches with the phone screen off, an aggressive battery saver, or an OS OOM), the affected diary items are lost on next launch. Water is unaffected. Open incident tracked upstream; workaround is to switch dates or background the app briefly after a large log to force a flush.

## Related

- [Foods, meals & recipes](foods.md)
- [Goals & Adaptive TDEE](goals.md)
- [Trace in NutriTrace + Smart Log](trace.md)
- [Activity & Intermittent Fasting](activity-if.md)
