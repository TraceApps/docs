# Migrate from Mealime

Mealime shuts down on October 21, 2026. Mealime has no export of its own, but a free, unofficial browser extension saves your recipes to a zip that CookTrace imports as it is, photos and notes included.

## Export from Mealime

The [Recipe Exporter for Mealime](https://github.com/ianknauer/mealime-recipe-exporter) is a Chrome extension made by BellaNora. It isn't part of CookTrace or Mealime. It runs in Chrome, Edge, Brave and other Chromium browsers on a computer, not on a phone.

1. Install it from the [Chrome Web Store](https://chromewebstore.google.com/detail/recipe-exporter-for-meali/fjgiehibnolnihlanldlbcgjmcghlilc), or load it by hand as its README describes.
2. Sign in at [my.mealime.com](https://my.mealime.com) (or app.mealime.com).
3. From that tab, click the extension's icon, then **Export my recipes**. The zip (`mealime-recipes-<date>.zip`) is saved to your downloads.

What it saves depends on which Mealime site you export from: my.mealime.com gives your favorites, the recipes in your current and previous meal plans, recipes you created yourself, and your recipe notes. app.mealime.com gives your favorites only. The extension's README has the full table.

## Import into CookTrace

Don't unzip it. In **Settings**, open **Import from Another App**, tap **Choose File** and pick the zip. CookTrace finds one recipe per file in the zip's `schema-org` folder (the other files, `recipes.json`, `raw` and `markdown`, hold the same recipes and aren't imported twice), shows them in the picker, and imports the ones you tick.

The Android app imports the same zip, with or without a server.

## What carries over

- Name, photo, servings, cook time, ingredients, steps and cookware (as tools).
- Nutrition per serving: calories, fat, saturated and trans fat, cholesterol, sodium, carbohydrates, fiber, sugars and protein.
- Your notes on a recipe, in the recipe's notes.
- For a recipe you added to Mealime from a website, the link to that page, as the recipe's source.

A recipe you already have (same name, or same source link) is skipped, as with every import. See [the dedup policy](import.md#dedup-policy).

## What doesn't

- **The meal plan's days.** Recipes from your meal plan come in, but not which day they were on. Plan them again in the [Cook Diary](diary.md), then build the list with [Add from Planned Cooks](shopping.md#add-from-planned-cooks).
- **Shopping lists.** The extension doesn't export them.
- **Which ingredients each step uses.** Mealime lists them per step; the steps come in as text.
- **Amounts written around a package size**, such as "½ (227 g) block tofu", come in as written: the ½ as the amount and "227 g" as a note. Check them once if you want the pantry to link them.

## Thumbnails in the picker

The picker shows a recipe's photo only when it's small enough to send with the list (64 KB). Mealime's photos are usually larger, so the picker shows placeholders. The photos still import with each recipe.

## Related

- [URL, file, and photo import](import.md)
- [Migrate from Mealie](migrate-mealie.md)
- [Migrate from Paprika](migrate-paprika.md)
- [Migrate from Tandoor](migrate-tandoor.md)
