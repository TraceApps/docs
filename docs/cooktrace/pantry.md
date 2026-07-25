# Pantry

The Pantry is the catalog of ingredients you actually have on hand. Recipe ingredient rows can link to a pantry entry so the app knows what's in stock, what's expiring, and what needs to land on the shopping list.

## Layout

Grid or List view, per-user preference. Cards show hero image, name (with brand suffix on variants), stock pill, category chip, and an "Expiring Soon" chip when the item is inside the digest window.

Tap any item to open the **slide-up details sheet**. It covers the lower half of the screen with:

- Hero and brand.
- Category.
- Barcode (if scanned).
- Stock pill (In Stock / Out / Low).
- Serving size and on-hand quantity.
- Expiration date.
- Nutrition table (respects your visible-nutriments picker under Settings > Nutrition).

Hit **Edit** and the same sheet swaps its display rows for input fields. No separate editor screen, no route change. Save writes back and the sheet re-renders in display mode.

## Variants

Pantry entries have three shapes:

1. **Flat**: a single row, no children (e.g. "Baking Soda").
2. **Generic parent**: a row that represents the concept ("Milk") with no stock of its own; brand children carry stock.
3. **Variant**: a child ("Oatly Barista") under a generic parent. Its stock and nutrition are per-variant.

Recipe ingredients link to the parent when possible. Any in-stock variant satisfies the pantry-match check, so the recipe card's "8/10 in pantry" pill counts "Milk" as covered when Oatly Barista is on the shelf.

Delete a generic parent and CookTrace prompts **Keep Variants** (promote each variant to flat) vs **Remove All**. Three-level nesting is rejected at the server, per `src/lib/pantry-variants.js`.

## Search behaviour

Pantry search picks the right thing depending on how specific your query is:

- Query matches a **standalone variant** exactly (brand plus product) and only that variant is returned.
- Query matches a **generic parent** and no variant precisely: the parent is returned, expanded to show its variants inline.
- Query is loose: all matches surface, ranked parent-first.

This means typing "oatly" in a Recipe Editor's ingredient picker jumps straight to the Oatly Barista row, while typing "milk" lets you pick which milk you meant.

## Expiration dates and digest

Set an **Expires On** date when adding or editing an item. Cards then show:

- **Expiring Soon** chip when today is inside the digest window.
- **Expired** chip when the date has passed.

The **Expiration Digest** notification bundles everything expiring in the next N days into a single push. Configure under Settings > Notifications:

- **Enable expiration digest** master toggle.
- **Days before**: 1, 3, 5, 7, or 14.
- **Delivery time**: `HH:MM` in the container timezone.

Delivery uses whatever push provider you have configured (Apprise, Gotify, or ntfy). The scheduler runs a boot tick 5 seconds after startup, then every 15 minutes, and deduplicates per `(user, kind, ref_id, fired_date)` in the `notification_log` table so you never get two of the same digest in one day.

## Inline nutrient editing

The details sheet's Nutrition rows are editable directly in Edit mode. Pick a serving size (e.g. `100 g`, `1 slice (28 g)`) and enter values per serving; the FDA-style label on any recipe using this item recomputes on the fly.

For **Recompute From Pantry** to cross the volume-to-mass boundary, each item can carry a **`g_per_cup`** density override. Recipe Editor and Recipe View both surface a "Set N g/cup" chip next to any skipped ingredient so you can fill densities in one tap without leaving the recipe.

## Barcode scanning

The scanner is native (ML Kit) on Android via `@capacitor-mlkit/barcode-scanning` and QuaggaJS in the browser. Under Settings > Connected Services > Barcode Scanner:

- **Beep on Scan** (default on).
- **Auto-Enable Flashlight** (web only; Android's system camera handles torch).

A scan looks the code up in this order:

1. Local pantry (returns the existing item if found).
2. Open Food Facts (if enabled).
3. USDA FoodData Central (if enabled and an API key is set).

The result pre-fills a new pantry entry, which you can accept, tweak, or discard.

## Open Food Facts

Free, no key. Enable under Settings > Connected Services > Open Food Facts. Optional fields:

- **Search Language** and **Search Country** for locale-appropriate results.
- **Upload Country** for the "Share to OFF" flow.
- **Account Username / Password**: needed only if you want to upload contributions back to OFF.

v1.0.1 added an All-mode search that pulls from every regional OFF instance, source pinning, tier filters, origin badges, and a data-completeness dot so you can prefer richer records at a glance.

## USDA FoodData Central

Optional. Grab a free API key from `https://fdc.nal.usda.gov/api-key-signup`, paste it into Settings > Connected Services > USDA. Search results now carry a data-type badge (Foundation, SR Legacy, Branded, Survey (FNDDS)) so you can tell what kind of entry you're picking. Per-user, so different household members can use different keys or none at all.

## Pantry categories

Server-managed per user. Fifteen defaults seed on first boot (`server/db.js:506`), each with an icon, colour, and **default aisle**. The shopping list uses `default_aisle` to auto-populate an item's aisle when you send ingredients from a recipe.

Manage them under **Manage > Pantry Categories** (drag to reorder, add, rename, recolour, or delete).

## Auto-add ingredients

Off by default. When on, saving a recipe creates any missing pantry rows for its ingredient list, name-matched case-insensitively. Handy when you're bulk-importing a Mealie backup and want the pantry to catch up.

## Related

- [Recipes](recipes.md)
- [Shopping in Feature tour](features.md#shop)
- [Trace AI in CookTrace](trace.md)
- [Settings reference](settings.md)
