# Pantry

The Pantry is the catalog of ingredients you actually have on hand. Recipe ingredient rows can link to a pantry entry so the app knows what's in stock, what's expiring, and what needs to go on the shopping list.

## Layout

Grid or List view, per-user preference. Cards show hero image, name (with brand suffix on variants), stock pill, category chip, an "Expiring Soon" chip when the item is inside the digest window, and how many recipes use the item. Tap that number to open the item's sheet on its **Used In** list.

Tap any item to open the **slide-up details sheet**. It covers the lower half of the screen with:

- Hero and brand.
- Category.
- Barcode (if scanned).
- An **In Stock** switch. Flip it right there, no need to enter Edit: back in stock keeps the amount you have (or sets 1), out of stock sets 0. A generic item's stock is its variants', so it shows a summary instead.
- On Hand with **-** and **+**, and the serving size. Reaching 0 marks the item out of stock; a blank amount stays blank (untracked) until you tap +.
- Expiration date.
- Nutrition table (respects your visible-nutriments picker under Settings > Nutrition).
- **Used In**: every recipe that uses the item, with its photo. Tap one to open it; back returns to the Pantry with the sheet open where you left it. Long lists show six with a Show All. For a variant, the recipes that ask for its generic follow under "Also Through ..."; for a generic, the ones that ask for a particular variant follow under "Through Its Variants". Useful before renaming, merging or deleting an item.

Hit **Edit** and the same sheet swaps its display rows for input fields. No separate editor screen, no route change. Save writes back and the sheet re-renders in display mode.

A link to `#/pantry/<id>` (an old bookmark, or **Open in Pantry** from a shopping list item) opens the Pantry with that item's sheet up.

## Variants

Pantry entries have three shapes:

1. **Flat**: a single row, no children (e.g. "Baking Soda").
2. **Generic parent**: a row that represents the concept ("Milk") with no stock of its own; brand children carry stock.
3. **Variant**: a child ("Oatly Barista") under a generic parent. Its stock and nutrition are per-variant.

Recipe ingredients link to the parent when possible. Any in-stock variant satisfies the pantry-match check, so the recipe card's "8/10 in pantry" pill counts "Milk" as covered when Oatly Barista is on the shelf.

Delete a generic parent and CookTrace prompts **Keep Variants** (promote each variant to flat) vs **Remove All**. Three-level nesting is rejected at the server, per `src/lib/pantry-variants.js`.

## Search behavior

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
2. Open Food Facts / OFF (if enabled).
3. USDA FoodData Central (if enabled and an API key is set).

The result pre-fills a new pantry entry, which you can accept, tweak, or discard.

Long-press the app icon and choose **Scan to Pantry** to open the Pantry with the scanner already open (see [App icon shortcuts](shopping.md#app-icon-shortcuts)).

## Open Food Facts

Free, no key. Enable under Settings > Food Sources > Open Food Facts. Optional fields:

- **Search Language** and **Search Country** for locale-appropriate results.
- **Upload Country** for the "Share to OFF" flow.
- **Account Username / Password**: your own Open Food Facts account, needed for **Share to OFF**.

**Share to OFF** in a pantry item sends an item that isn't in Open Food Facts yet, with its photo, under your name. It needs your own Open Food Facts account, which is free: create one at [openfoodfacts.org](https://world.openfoodfacts.org/), then enter its username (not your email) and password under **Settings > Food Sources > Open Food Facts**. Without them, Share to OFF asks for them right there in the item, keeping what you typed, then shares. The item goes through your CookTrace server (the Android app with no server sends it to Open Food Facts itself), and a wrong username or password, an item OFF refuses, or OFF being unreachable each get a plain message.

v1.0.1 added an All-mode search that pulls from every regional OFF instance, source pinning, client-side tier filters, and the shared **quality signals** (origin-country flag + completeness dot on every OFF result). See [Food data quality signals](../reference/food-data-quality.md) for what each signal means and how to filter on it.

## Allergens

A pantry item keeps the allergens on its label: what it **Contains** and what it **May Contain**. Scanning a barcode or downloading a product from Open Food Facts fills them in, marked **From the label**. **Edit** in the item's Allergens box lets you set them yourself (a store brand, a product Open Food Facts doesn't have), and the box then says **You set this**. An item nobody has set says **Not known yet**.

A recipe that uses the item picks these up, so a gluten-free flour linked to the recipe makes the recipe gluten free, where the word "flour" alone would not. Allergens that are a problem for someone in your [household](settings.md#household-per-user) name them ("Gluten · Sam").

## USDA FoodData Central

Optional. Grab a free API key from `https://fdc.nal.usda.gov/api-key-signup`, paste it into Settings > Connected Services > USDA. Results carry a **data-type badge** (Foundation, SR Legacy, Branded, Survey (FNDDS), Experimental) so you can tell what kind of entry you are picking. Per-user, so different household members can use different keys or none at all. See [Food data quality signals](../reference/food-data-quality.md) for how to read the badges.

Items are saved per 100 g (100 ml for drinks), the way USDA gives every value. To bring an item added from USDA up to date, tap **Edit** on it and then **Refresh from USDA**, under the barcode. It needs USDA turned on with your API key, and a connection. It fills the item's nutrition and serving from USDA the way Refresh from OFF does, and nothing is saved until you tap Save. Earlier versions saved an item added from USDA with a serving size other than 100 g with wrong values ([nutritrace#290](https://github.com/TraceApps/nutritrace/issues/290)); Refresh from USDA, or adding the item again, fixes it; a recipe's saved nutrition picks up the new values when it's calculated again.

## Pantry categories

Server-managed per user. Fifteen defaults seed on first boot (`server/db.js:506`), each with an icon, color, and **default aisle**. The shopping list uses `default_aisle` to auto-populate an item's aisle when you send ingredients from a recipe.

Manage them under **Manage > Pantry Categories** (drag to reorder, add, rename, recolor, or delete).

## Auto-add ingredients

Off by default. When on, saving a recipe creates any missing pantry rows for its ingredient list, name-matched case-insensitively. Handy when you're bulk-importing a Mealie backup and want the pantry to catch up.

## Related

- [Recipes](recipes.md)
- [Shopping in Feature tour](features.md#shop)
- [Trace AI in CookTrace](trace.md)
- [Settings reference](settings.md)
