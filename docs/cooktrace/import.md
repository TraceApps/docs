# URL, file, and photo import

CookTrace's import surface is deliberately broad because the recipes you already care about live in ten different places. This page covers what each entry point does, the three URL-import tiers, the bulk-zip flow, and the deduplication rules.

## URL import: three tiers

Under Settings > Cooking > URL Import Engine, pick one:

### Standard

The default. Fetches the URL server-side and parses **schema.org/Recipe JSON-LD**, which is what the vast majority of food blogs and publisher sites already publish for Google. Fast, no external dependencies, no AI quota consumed.

- Implementation: `server/lib/recipe-scraper.js`.
- Hardening: SSRF-guarded (http(s) only, private IP block, 8 second timeout, 5 MB response cap, custom User-Agent).
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
- Requires a configured AI provider under Settings > AI Assistant.
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

## Photo import

The `PhotoImportDialog` (also accessible via the `create_recipe` tool that Trace calls) takes a picture of a printed recipe, a handwritten card, or a phone screenshot and returns a filled-in recipe you can review before saving. Trace does the OCR and structuring in one pass, so you need a configured AI provider.

The system prompt is tuned to preserve ingredient groupings ("Dough" vs "Sauce"), pick up per-step timing, and keep quantities as-written rather than rounding.

## Bulk zip import

For whole libraries (a Mealie backup, a Paprika archive, a folder of exported files), use **Bulk Import** under Settings > Import from Another App.

The flow is two-step:

1. **Scan**: upload the zip. The server unpacks it into `.import-cache/<uuid>/` and returns a list of recipes it found, each with a thumbnail if one was in the archive.
2. **Commit**: pick which to import (all by default) and confirm. The server writes them into your recipe library, moving images from the cache into `uploads/`.

Cancelling from the picker deletes the cache. Committing keeps the cache in `.import-cache/` for the session and cleans it up later; the cache is intentionally excluded from full-backup dumps and from restore.

### Dedup policy

The import endpoint (`POST /api/recipes/import`) takes `dedup: 'skip' | 'force'`.

- **`skip`** (default in the bulk UI): a case-insensitive name match **or** an exact `source_url` match against your existing recipes counts as a duplicate and gets skipped.
- **`force`**: import anyway, adding a second copy. Handy when you're intentionally re-importing an updated version.

The commit response reports how many landed, how many were skipped as dupes, and how many failed to parse.

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
