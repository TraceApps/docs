# Migrate from Paprika

Paprika 3 exports recipes as a `.paprikarecipes` file: a zip archive containing one `.paprikarecipe` per recipe, each usually a gzipped JSON payload. CookTrace's bulk importer accepts the archive directly and writes each recipe into your library.

## Export from Paprika

The steps vary slightly per platform (macOS / iOS / Windows), but the general path in Paprika 3:

1. Open Paprika.
2. Go to **File > Export Recipes** on macOS/Windows, or use the equivalent share sheet on iOS.
3. Pick **Paprika Recipe Format** (`.paprikarecipes`).
4. Choose whether to export the whole library or just a selection.
5. Save the resulting `.paprikarecipes` file somewhere you can upload from.

The file is a normal zip, so if you're curious you can rename it to `.zip` and peek inside; you'll see one `.paprikarecipe` per recipe. The importer copes with both the standard gzipped payload and the (rarer) plain-JSON variant Paprika sometimes emits.

## Import into CookTrace

Under **Settings > Import from Another App > Bulk Import**, drop the `.paprikarecipes` file (or click to pick). The server unpacks it into `.import-cache/<uuid>/` and returns a picker of every recipe it found, with a thumbnail if the recipe carried one.

Two-step flow:

1. **Scan**: upload only. Nothing is written to your library yet. Review the picker.
2. **Commit**: tick the recipes to import (all by default) and confirm. The server writes them into your recipe library and moves images from the cache into `uploads/`.

The commit response reports how many landed, how many were skipped as duplicates, and how many failed to parse. See [URL, file, and photo import](import.md#dedup-policy) for how the duplicate check works.

### Import as a Cookbook

CookTrace also has a **CookbookImportDialog** flow that treats a `.paprikarecipes` archive as a single Cookbook: every recipe in the archive lands as usual, plus a new [Cookbook](cookbooks.md) is created containing all of them. Handy when your Paprika library is already organised by cookbook and you want to preserve the grouping instead of dumping everything flat into your library.

## What carries over

Per `_importPaprika` in `server/lib/recipe-importers.js`:

- **Name**, **description**, **notes**.
- **Ingredients**: parsed from Paprika's newline-separated `ingredients` string into structured qty / unit / name / note rows.
- **Steps**: parsed from Paprika's `directions` field on blank-line boundaries; each paragraph becomes one step.
- **Servings** and yield text.
- **Prep** and **cook** times (Paprika stores free-form strings like "1 hr 15 min"; CookTrace parses them into minutes).
- **Photo**: `photo_url` if present, otherwise the base64-encoded `photo` field.
- **Category**: the first entry in Paprika's `categories` array lands on the recipe's Category slot. Remaining entries (up to 12) become tags.
- **Source URL** (`source_url` or older `source`).
- **Created date**: `created` or the older `dateAdded` is preserved, so imported recipes keep their original creation timestamp instead of stamping to today.

## What doesn't

Paprika keeps **meal plans** and **grocery lists** in a separate export path that CookTrace doesn't ingest. Rebuild the meal plan directly in the [Cook Diary](diary.md) and let [Shop This Plan](shopping.md#shop-this-plan) regenerate the grocery list from the pantry-aware endpoint.

Paprika **bookmarks** and **cookmarks** (the "I cooked this" history) also don't round-trip. If you want the cook history preserved, hand-log a few of the most recent cooks in the Cook Log after import; going deeper isn't worth it in practice.

## Size cap

`IMPORT_ZIP_MAX_MB` defaults to **512** MB. Bump it in your environment if your archive is bigger, or drop it if you want a stricter guard rail. See [env vars](env-vars.md).

## Related

- [URL, file, and photo import](import.md)
- [Migrate from Mealie](migrate-mealie.md)
- [Migrate from Tandoor](migrate-tandoor.md)
