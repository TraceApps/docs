# Recipes

Everything you can do with a single recipe. For the higher-level tour of the Recipes page and how it fits with the rest of CookTrace, see [Feature tour](features.md).

## The recipe model

Each recipe carries:

- **Name** (required) and optional **description**.
- **Hero image** (`img_url`), uploaded or pasted URL.
- **Star rating** (1-5, or unrated) and **favorite** heart.
- **Yield / servings** (`servings`, default 2).
- **Prep**, **cook**, **rest**, and **total** times, all in minutes. Total auto-calculates when blank.
- **Ingredients**, grouped into named sections (e.g. "Dough", "Sauce") with per-row quantity, unit, ingredient text, and an optional link to a pantry item.
- **Steps**, ordered, with per-step text and optional per-step photo.
- **Kitchen gear** (`tools`) as a chip list, icon-mapped where recognised.
- **Source URL** (where you imported it from) and optional **video URL**.
- **Notes**, rich-text with bold, italic, underline, strikethrough, colour, bulleted and numbered lists. HTML sanitised on save.
- **Category**, **tags**, **cookbooks** it belongs to.
- **Cook history**, updated whenever a cook is logged.

### Rest time and Total time

Rest is its own slot (bread proof, marinade, dough chill) so it rolls into Total without hiding inside Cook. If Total is left blank on save, it auto-calculates as `prep + cook + rest`. If you type your own Total, that value wins. Any of the four can stay empty.

### Per-step photos and rich text

Each step has a camera button that attaches a photo taken or picked on device. In step text, `**bold**`, `*italic*`, and `__underline__` render inline. Duration mentions ("15 minutes", "1h 20m") get parsed into inline timer chips you can tap to start; parsing lives in `src/lib/parseTimes.js`.

## Live recipe scaling

Chips above the ingredient list let you scale ×0.5 / ×1 / ×2 / ×3, or set a custom serving count. Quantities re-render as clean mixed fractions (`¼ cup`, `1½ tsp`) using the snapper in `src/lib/qty.js`, which picks cooking-friendly denominators (halves, quarters, thirds, eighths). Scaling is view-only; the saved recipe is untouched.

## Inline unit converter

Each ingredient row has a small unit converter driven by:

- A 37-unit catalog in `src/lib/units.js` (mass, volume, count, plus common cooking units like cups, tablespoons, sticks).
- A 250-entry density table used for volume-to-grams conversions.

Custom units are addable per user under Manage > Units. Built-in units can be disabled (grayed out from pickers) without deletion.

## Cook Mode

Cook Mode is a distraction-free view with larger type, sticky ingredients, and the timer rail inline. It requests the **Screen Wake Lock** so the display stays on, and re-acquires the lock on `visibilitychange` when you tab back in.

Ingredient and step checkboxes persist to `localStorage` as `ct:checks:<recipeId>:ing|step` and are **intentionally device-local**. Different devices at different stations (phone at the counter, tablet above the stove) keep independent progress.

## Cook log

Log a cook from Cook Mode or from the recipe view. Each entry has:

- Date (defaults to today, backdatable).
- Meal type (Breakfast / Lunch / Dinner / Snack / null).
- Per-cook rating (separate from the recipe's overall rating; answers "how did it turn out this time").
- Notes.
- Optional photos.

The recipe view shows a **per-recipe cook history** so you can flip back to what you tweaked last time. `last_cooked_at` and cook count roll up on the card.

## FDA-style Nutrition Facts box

The Nutrition column renders a full FDA-style label via `NutritionFactsBox.svelte`. Thirty-one nutriments in real label order, %DV column, sodium and salt auto-derived from each other (factor 400), servings and serving-size at the top. A per-user picker under Settings > Nutrition controls which rows the label shows inline; the rest still count for totals.

### Auto-recompute from Pantry

Tap **Recompute From Pantry** on the Nutrition column and every linked pantry row contributes its nutrition to the total, weighted by ingredient quantity. Volume-to-grams jumps use the density table and, cross-family (mass to volume), each pantry item's `g_per_cup` override. Ingredients without a linked pantry item or without a density get a skipped-item badge with an inline "Set N g/cup" chip so you can fill the gap in one tap.

## Sharing

Three separate sharing surfaces:

- **Per-user grants**: `POST /api/recipes/:id/shares` adds a grantee by user id. The recipe surfaces on their Shared tab.
- **Kitchens fan-out**: sharing a recipe with a Kitchen mints one grant per member, tagged `via_kitchen_id`. Removing a member (or leaving the Kitchen) revokes only grants tagged that way. See [Kitchens](kitchens.md).
- **Public share link**: `POST /api/recipes/:id/share` mints a random URL-safe token. The recipe is then readable without login at `/r/<token>`. Bypasses the setup-required gate so a fresh install serving a shared link doesn't 503. `DELETE /api/recipes/:id/share` revokes.

Card image: `GET /api/recipes/:id/card.png` returns a server-rendered Pinterest-style card for social sharing.

## Long-press and print

Long-press any recipe card for a context menu with Cook Now, Add to Cookbook, Share, Print, and Delete. The recipe view has a print stylesheet via `@media print` that strips chrome, sidebars, and interactive controls for a clean paper copy.

## Comments

Flat plus threaded comments per recipe. Anyone with read access can post; authors and admins can edit or delete. Reply notifications go out via the configured push provider when `notifRecipeComments` is on (default on).

## Related

- [Feature tour](features.md)
- [Pantry](pantry.md)
- [Cook Diary and Meal Planner](diary.md)
- [Import from URL, file, or photo](import.md)
- [Trace AI in CookTrace](trace.md)
