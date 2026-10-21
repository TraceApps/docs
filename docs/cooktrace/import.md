# URL, file, and photo import

CookTrace's import surface is deliberately broad because the recipes you already care about live in ten different places. This page covers what each entry point does, the three URL-import tiers, the bulk-zip flow, and the deduplication rules.

## URL import: three tiers

Under Settings > Cooking > URL Import Engine, pick one. An AI agent can run the same import over [MCP](mcp.md) with `import_recipe_from_url`, using your choice here or a tier it names.

### Standard

The default. Fetches the URL server-side and parses **schema.org/Recipe JSON-LD**, which is what the vast majority of food blogs and publisher sites already publish for Google. Fast, no external dependencies, no AI quota consumed.

- Implementation: `server/lib/recipe-scraper.js`.
- Hardening: SSRF-guarded (http(s) only, 8 second timeout, 5 MB response cap, custom User-Agent). Addresses on your own network (directly, through a name, or through a redirect) are open only to an admin; `ALLOW_PRIVATE_RECIPE_URLS=1` opens them to every account and to a single-user install (from the next release).
- Falls through cleanly when a site doesn't emit JSON-LD, in which case the parser returns an empty result and the UI offers to try Enhanced or Smart.

Use it when the URL is from a mainstream recipe site.

### Enhanced

Same as Standard, plus a shell-out to the Python **`recipe-scrapers`** library (300+ site-specific extractors, `>=14.0.0`). Baked into the Docker image at build time, so no extra setup on your side.

- Implementation: `server/lib/recipe-scrapers-bridge.js`, `spawn(python3, ...)`.
- Python binary is configurable via `PYTHON_BIN` (default `python3`).
- Handles sites where JSON-LD is malformed, missing, or where category, tag, and yield extraction needs site-specific rules.
- The **Enhanced Fallback** setting picks what to try if the recipe-scrapers extractor fails: Standard or Smart.

Use it when Standard misses fields or the site is quirky.

!!! warning "Enhanced needs the server"
    Enhanced is silently unavailable in Android's local (offline) mode; the setting UI disables the option because there's no Python bridge to shell to. Connected mode uses the server's Python as expected.

### Smart

Passes the fetched page (or raw text) to your configured Trace AI provider with a tight system prompt asking for structured recipe JSON. Reserved for messier sources: sites that hide the recipe behind heavy JS, blog posts where the recipe is written prose-style, or plain text you've copied out of a book.

- Implementation: `server/lib/recipe-ai-fallback.js`.
- Requires a configured AI provider under Settings > AI Assistant, or Trace set up on the server (`AI_ENABLED`, `AI_PROVIDER`; from the next release also an OpenAI-compatible endpoint with `AI_PROVIDER=oai-compat`, with or without an API key).
- An AI address an account sets in its own settings is checked first: other accounts can't point it at your own network unless `ALLOW_PRIVATE_AI_URLS=1`, and a failed request shows its status and the provider's message.
- Uses your provider quota / local model.

Use it when Enhanced returns garbage or the source is unstructured.

## File and paste import

Under Settings > Import from Another App, or from the Recipes page's **Import** button, drop a single file or paste text. Auto-detects:

- **Mealie** JSON export.
- **Tandoor** JSON export.
- **Paprika** `.paprikarecipes` archive (single or multi-recipe).
- **schema.org/Recipe** JSON-LD paste.
- **Saved HTML** upload (JSON-LD embedded in the page).
- **CookTrace** JSON export (portable format).
- **Plain text** paste (paragraphs become stub steps you can clean up in the editor).

Nothing to configure per format; the detector inspects the payload and picks the right parser.

## How ingredient lines are read

Every importer splits each ingredient line into an amount, a unit, a name and a note, the same way on the server and in the Android app:

- **Amounts**: whole numbers, decimals, fractions and mixed numbers (`1 1/2`, `½`, `1½`), and ranges (`4 - 5`, `1 to 2`), kept as one amount.
- **Units**: spelled out or abbreviated, singular or plural (`tablespoons`, `Tbsp.`, `cups`), stored as the short form (`tbsp`, `cup`).
- **The note** gets what describes this batch rather than the ingredient: anything in parentheses, sizes (`large`, `medium`), package sizes (`15-ounce`, `14.5 oz`), packing (`packed`) and prep after a comma (`garlic, minced`). A comma that's part of the name stays (`boneless, skinless chicken thighs`).

The name is what links to the pantry. With **Link Ingredients to Pantry** on, each ingredient links to the pantry item of the same name, regardless of case or plural (`tomato` finds `Tomatoes`), and a new pantry item, out of stock, is made for any name the pantry doesn't have yet.

## Photo import

The `PhotoImportDialog` (also accessible via the `create_recipe` tool that Trace calls) takes a picture of a printed recipe, a handwritten card, or a phone screenshot and returns a filled-in recipe you can review before saving. Trace does the OCR and structuring in one pass, so you need a configured AI provider.

The system prompt is tuned to preserve ingredient groupings ("Dough" vs "Sauce"), pick up per-step timing, and keep quantities as-written rather than rounding.

## Bulk zip import

For whole libraries (a Mealie backup, a Tandoor export, a Paprika archive, a [Mealime export](migrate-mealime.md), a folder of exported files), open **Settings > Import from Another App** and **Choose File**.

The flow is two-step:

1. **Scan**: upload the zip. The server unpacks it into `.import-cache/<uuid>/` and returns a list of recipes it found, each with a thumbnail if one was in the archive.
2. **Commit**: pick which to import (all by default) and confirm. The server writes them into your recipe library, moving images from the cache into `uploads/`.

Canceling from the picker deletes the cache. Committing keeps the cache in `.import-cache/` for the session and cleans it up later; the cache is intentionally excluded from full-backup dumps and from restore.

### Dedup policy

A recipe counts as a duplicate when its name matches one of your recipes (ignoring case) **or** its source link matches exactly. That includes a recipe imported earlier in the same archive, so an export that holds the same recipe twice imports it once.

The bulk import's **If a Recipe Already Exists** option decides what happens (`dedup` on `POST /api/recipes/import-zip/commit`):

- **Skip It** (`skip`, the default): the duplicate is left out and your recipe stays as it is.
- **Replace with the Imported Version** (`replace`): your recipe is updated from the archive.
- **Allow Duplicates** (`force`): import anyway, adding a second copy.

After a bulk import, the summary reads "Imported 119 of 121 Recipes" when anything was left out. It lists the skipped duplicates by name (each links to the recipe it matched) and, separately, any recipe that failed with the reason. To bring skipped ones in anyway, set the option to Allow Duplicates and import the file again.

Single imports (a URL, pasted JSON or HTML, or one uploaded file) always skip a duplicate. You get "This recipe already exists. Opening it." and the existing recipe opens. A Paprika archive uploaded this way reports how many it imported and how many duplicates it skipped.

## Size cap

`IMPORT_ZIP_MAX_MB` (default **512** MB) caps the bulk upload. Bump it in your environment if you regularly import bigger libraries, or drop it if you want a stricter guard rail. See [env vars](env-vars.md).

## Cookbook archive import

`CookbookImportDialog` handles a **Paprika** `.paprikarecipes` archive as a Cookbook: every recipe in the archive lands in your library and a new Cookbook is created holding all of them, preserving the source grouping.

## NutriTrace foods to Pantry

Separate from recipe import, `SettingsImportFromNT.svelte` (under Settings > NutriTrace) pulls foods from your NutriTrace instance into the Pantry. Requires the [NutriTrace federation](../integrations/federation.md) URL and token to be set. One-time backfill in the usual case.

## Portable JSON export and import

Under Settings > Backup & Restore:

- **Export JSON** dumps everything the API can round-trip (recipes, pantry, categories, tags, cookbooks, settings), no images. Payload cap 25 MB on `POST /api/data/import`.
- **Import JSON** merges into the current account (does not wipe).

For whole-instance moves (including images), use **Full Backup** instead. That path zips DB and uploads together and is admin-only. See the [backups guide](../self-hosting/backups.md).

## Related

- [Migrate from Mealie](migrate-mealie.md)
- [Recipes](recipes.md)
- [Trace AI](trace.md)
- [Env vars](env-vars.md)
