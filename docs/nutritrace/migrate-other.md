# Migrate from Lose It, Cronometer, Waistline

Three community trackers, three different export shapes, one import flow inside NutriTrace. All of them land in the same tables via **Settings → Import & Export → Import from another app**, with a preview step and per-date conflict policy before anything writes.

For MyFitnessPal specifically, see [Migrate from MyFitnessPal](migrate-mfp.md); it has enough quirks to earn its own page.

## Preview and conflict policy (all three sources)

Every commit goes through `POST /api/nutrition-import/preview` first (`server/routes/nutrition-import.js`). The preview returns:

- Every food and diary row that would be inserted.
- Per-date grouping so you can see what lands where.
- Dupe detection against your existing catalog by name and brand.
- Parse warnings for rows that skipped or transformed.

Only after you confirm does the commit endpoint (`POST /api/nutrition-import/commit`) actually write. The **per-date conflict policy** applies at commit time:

- **Skip**: leave any date that already has diary data untouched.
- **Merge**: add imported rows alongside anything already there.
- **Overwrite**: replace whatever is on that date with the imported rows.

The UI groups dates so you can, for example, skip everything before 2024 and overwrite everything after.

## Lose It

??? details "Lose It (CSV, subscription required to export)"
    Lose It gates its data export behind **Lose It Premium**. If you have a paid subscription:

    ### Export from Lose It

    1. Log in to [loseit.com](https://www.loseit.com/) on desktop.
    2. **Account → Data → Export**.
    3. Pick a date range and download the CSV. The export contains foods, exercises, and body measurements as separate CSV files inside a zip.

    Lose It's mobile app does not surface the export; use the web app.

    ### Import into NutriTrace

    1. Open **Settings → Import & Export → Import from another app**.
    2. Pick **Lose It (CSV)** from the source picker.
    3. Upload the foods CSV first (this brings across your custom Lose It foods into your Local catalog).
    4. Upload the diary CSV. **Preview** shows the days and rows it would insert.
    5. Pick a conflict policy per date range and **Commit**.

    ### What survives

    - **Foods**: name, brand, per-serving macros, serving size, category (mapped to your NutriTrace categories where possible).
    - **Diary rows**: per-food entries with quantity, unit, meal slot. Lose It's meal slots (Breakfast / Lunch / Dinner / Snacks) map directly to the NutriTrace defaults; if you have renamed your slots, the importer surfaces a slot mapping step.
    - **Body measurements**: weight, waist, and any custom fields land under Diary → Body Stats on the appropriate dates. Backdated entries are accepted.

    ### What doesn't

    - **Recipes**: Lose It's recipe export is opaque JSON that lists ingredients as free text without linked food IDs. Re-create high-value recipes manually in NutriTrace's Recipe editor.
    - **Photos**: no image export from Lose It.

## Cronometer

??? details "Cronometer (CSV, free)"
    Cronometer exports for free. No subscription needed.

    Cronometer offers two exports with diary data in them, and which one you need depends on what your account gives you.

    ### Try Food & Recipe Entries first

    1. Log in to [cronometer.com](https://cronometer.com/) on desktop.
    2. Open your **Account** page, scroll to **Account Data**, and click **Export Data**.
    3. Pick a date range, then click **Export Food & Recipe Entries**. It downloads a file named `servings.csv`, with one row per food you logged, which is why older guides call it the Servings export.
    4. Open it and look at the columns. If it has `Energy (kcal)` and the other nutrient columns, that's the one to import: it gives you every food with its own nutrition.

    On free accounts Cronometer has stopped putting the nutrition in this file, leaving only `Day, Time, Group, Food Name, Amount, Category`. If that's what you have, there is nothing in it to import, and NutriTrace will say so and point you to the other export.

    ### Otherwise use Daily Nutrition

    1. In the same **Export Data** dialog, tick **Include diary group rows**.
    2. Click **Export Daily Nutrition**. It downloads `dailysummary.csv`.

    This one always has the numbers, but as totals rather than per food. With the group rows ticked you get one row per meal per day, which imports as one entry in each of your meals. Without it you get one row per day, which imports as a single entry per day in your first meal.

    ### Import into NutriTrace

    1. **Settings → Import & Export → Import from another app**.
    2. Pick **Cronometer (CSV)**.
    3. Upload the file. **Preview**, pick a conflict policy, **Commit**.

    The file name doesn't matter. NutriTrace works out which export it is from the columns, so a renamed file imports the same way.

    ### What survives

    - **From Food & Recipe Entries**: one diary entry per food, with the amount you logged ("150.00 g", "1 cup", "2 each"), the time including AM and PM, and the nutrition Cronometer recorded for it. Zero-calorie entries such as supplements and most spices come across too, with their vitamins and minerals.
    - **From Daily Nutrition**: one entry per meal per day (or per day), named "Cronometer total", carrying that row's calories, macros and micronutrients.
    - **The meal it was in.** Cronometer's diary groups map to your NutriTrace meals by name, including `Uncategorized`. Anything that doesn't match goes to your last meal, and the preview lists those names before you commit.
    - **Dates in any order.** Cronometer writes them in your account's format, so NutriTrace reads the file to tell `9/4` (September 4th) from `9/4` (April 9th) rather than assuming.

    ### What doesn't

    - **Cronometer's other exports.** Only these two are read. Biometrics (weight, body measurements), Exercises, Notes and Fasts are not.
    - **Per-food detail from Daily Nutrition.** The file simply doesn't contain it. Totals are the most that export carries.
    - **Your Cronometer food library.** Imported entries go into your diary with their nutrition; they aren't added to your NutriTrace Foods.
    - **Recipes as recipes.** A logged recipe comes across as a diary entry, not as a NutriTrace recipe with ingredients.
    - **Brand as its own field.** Cronometer includes the brand in the food name ("Kirkland Signature, ..."), and it stays there.

## Waistline

??? details "Waistline (JSON, free, open-source)"
    [Waistline](https://github.com/davidhealey/waistline) is the open-source Android nutrition tracker NutriTrace grew out of (the storage prefix `wl_u<id>_<key>` is a Waistline holdover, per `ARCHITECTURE.md:193`). Waistline exports a full JSON backup with foods, meals, recipes, and diary rows all included. This is the highest-fidelity import path of the three on this page.

    ### Export from Waistline

    1. Open Waistline on Android.
    2. **Settings → Backup and Restore → Export**.
    3. Save the resulting JSON file (typically `waistline_backup.json`).

    Transfer the file to whatever device you are running NutriTrace's browser or Android app from.

    ### Import into NutriTrace

    Two entry points, depending on which NutriTrace surface you are on:

    - **Web / desktop**: **Settings → Backup and Restore → Restore Waistline backup**. Upload the JSON, preview, commit.
    - **Android**: same path. On Android in local-only mode, the import runs entirely on-device against the SQLite mirror.

    ### What survives

    - **Foods**: name, brand, barcode, per-serving and per-100g macros, categories. Photos are not carried over (Waistline stores them separately).
    - **Meals**: preserved as saved Meals.
    - **Recipes**: preserved as Recipes with ingredient linkage intact where the source ingredient food is in the export.
    - **Diary rows**: full per-food rows with quantity, unit, meal slot.
    - **Body stats**: weight and any custom Waistline fields.

    ### What doesn't

    - **Wellness data**: Waistline doesn't track it.
    - **Water containers**: Waistline's water tracking is coarser; NutriTrace's container library will need setting up under **Settings → Water**.

## Related

- [Migrate from MyFitnessPal](migrate-mfp.md)
- [Diary and meal logging](diary.md)
- [Foods, meals and recipes](foods.md)
