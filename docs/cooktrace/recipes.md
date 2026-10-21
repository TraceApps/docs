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
- **Kitchen gear** (`tools`) as a chip list, icon-mapped where recognized.
- **Source URL** (where you imported it from) and optional **video URL**.
- **Notes**, rich-text with bold, italic, underline, strikethrough, color, bulleted and numbered lists. HTML sanitized on save.
- **Category**, **tags**, **cookbooks** it belongs to.
- **Cook history**, updated whenever a cook is logged.

### Rest time and Total time

Rest is its own slot (bread proof, marinade, dough chill) so it rolls into Total without hiding inside Cook. If Total is left blank on save, it auto-calculates as `prep + cook + rest`. If you type your own Total, that value wins. Any of the four can stay empty.

### Per-step photos and rich text

Each step has a camera button that attaches a photo taken or picked on device. In step text, `**bold**`, `*italic*`, and `__underline__` render inline. Duration mentions ("15 minutes", "1h 20m") get parsed into inline timer chips you can tap to start; parsing lives in `src/lib/parseTimes.js`.

## Live recipe scaling

Chips above the ingredient list let you scale ×0.5 / ×1 / ×2 / ×3, or set a custom serving count. Quantities re-render as clean fractions (`1/4 cup`, `1 1/2 tsp`) for cups and spoons, picking cooking-friendly denominators (halves, quarters, thirds, eighths), and as decimals for grams and milliliters. Ranges scale at both ends (4-5 becomes 8-10). Scaling is view-only; the saved recipe is untouched.

Amounts written in the steps scale too, and are tinted so you can see which changed: "Whisk in 2 cups of milk" doubled reads "4 cups". Only measures scale (cups, spoons, grams, milliliters, ounces, pounds, and ranges of them). Times, temperatures, pan sizes and plain counts ("cut into 4 pieces") stay as written, and so does an amount for each piece ("2 tbsp of batter per pancake", "1/2 cup at a time"). The ingredients listed under a step scale with the rest.

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
- **Made From**: the version of the recipe it was made from (see [History](#history)). It's the recipe as it is when you log the cook; a cook logged for an earlier day takes the version the recipe had that day. If an older version was restored since then, the day can't tell which it was, so the cook asks you to choose. Change it when you logged the cook first and then edited the recipe to say what you really did.

The recipe view shows a **per-recipe cook history** so you can flip back to what you tweaked last time; each cook's **Made With Version 3** opens the recipe as you made it. `last_cooked_at` and cook count roll up on the card.

## History

Every change to what a recipe has you cook (its ingredients, steps, servings, yield or times) is kept as a version, and every cook remembers the one it was made from. Made for working on a recipe: change the flour, bake, compare, keep what worked. A new name, photo, note, tag or category isn't a new version, and neither is linking an ingredient to the pantry. An amount written another way (1/2, 0.5, ½) is the same version; any other change to it, however small (0.10 g to 0.15 g), is a new one.

**History** in the recipe's byline lists the versions, newest first. Each shows what changed from the one before ("Flour: 300 g → 250 g · Added sugar · Step 3 edited"), when, and how its cooks turned out (how many, the average rating, their photos). The current version and every version you cooked are shown; edits in between that you never cooked fold into a line ("3 more edits") until you open it.

A version opens its own page:

- The recipe as that version had it: ingredients, steps, servings and times.
- **Since then**: what's different in the recipe now. **Changed from Version 2**: what changed from the version before.
- **Compare with** the current recipe or the version before: ingredients lined up (a changed amount shows both), steps with the words that changed, and the servings and times that differ. Unchanged lines stay hidden until **Show Everything**.
- **Cooked From This Version**: each cook, its rating, notes and photos.
- **I Cooked This Version** logs a cook of it without changing the recipe, for when you bake the old base again.
- **Restore This Version** puts its ingredients, steps, servings and times back. The name, photo and notes stay as they are, and nothing is lost: what you had becomes a version too, and restoring an old version again makes it current rather than adding a copy.
- **Name This Version** gives it a name of its own ("Less flour", "The base"), shown in History, Compare and the cook dialog.
- **Delete Version**, only for a version no cook was made from.

Every version is kept, with no limit. Recipes from before 1.5 start their history with what they hold now; cooks logged before then say "Made before history was kept", because what they were made from isn't known. History works offline and on the Android app with no server, syncs like the rest, and is in backups, the Android app's local backup included. A cook logged offline keeps the version it was made from, and a version can't be lost to a cook: one deleted on another device before it heard of the cook comes back. Copies the app kept when two edits met (**Earlier Copies**) are at the bottom of History; see [Earlier versions](#earlier-versions). [Trace](trace.md) can read the history too: ask it what changed since your best batch, or which version you liked most.

### Earlier versions

When two devices change the same part of a recipe (say, the steps on your phone offline and in the browser), the newer edit stays and the other isn't lost (from the next release). The recipe page says **Two devices edited this recipe. The older edit is kept as an earlier version.**, with **View** and **Dismiss**.

**Earlier Versions** lists each copy kept: why it was kept (**Kept from a sync**, **Kept before a restore**, **Kept before a later save**, or someone's edit in a shared kitchen), **Differs in** (Title, Ingredients, Steps, Notes, Category, Photo, Nutrition, Details), and **Preview**, which shows its ingredients and steps. **Restore** asks **Restore This Version?** and makes it the recipe again; the recipe you had is kept as an earlier version, so a restore can be undone the same way. A kitchen member sees the earlier versions that were their own edits.

## Writing in Arabic, Hebrew and mixed lines

A recipe's name, description, ingredients, steps and notes each read the way their first letter does (from the next release). An Arabic ingredient sits on the right with its box, and its amount on the left; an Arabic step has its number on the right; and Arabic and English in one line come out in the right order. A new, empty line reads like the one above it. The recipe list, cookbooks, the editor, the shopping list, the pantry, the diary, the public recipe page, share cards and Trace's chat do the same. Only how a recipe is shown changes, not how it's saved. Hebrew and the other right-to-left scripts work the same way.

## Allergens

CookTrace works out what each recipe contains, using the EU's 14 allergens:

1. An ingredient linked to a pantry item uses that item's [allergens](pantry.md#allergens), from its label or set by you.
2. Otherwise the ingredient's name decides: "flour" reads as gluten, "soy sauce" as soy and gluten, "parmesan" as milk. Names that only look like one don't count: peanut butter isn't milk, eggplant isn't eggs, nutmeg isn't a nut, and "gluten-free flour" isn't gluten.
3. Your correction comes last. **Correct**, in the recipe's Allergens box, lets you mark what the recipe really contains, for what names and labels can't know.

Where it shows:

- **Recipe cards**: a warning for each allergen or diet that's a problem for someone in your [household](settings.md#household-per-user), naming them ("Peanuts · Alex"). Nothing else, so the cards stay quiet when there's nothing to watch for. **Safe for Everyone**, next to Favorites, keeps to recipes nobody has to avoid.
- **The recipe view**: under the title, everything it contains (warnings first, named), what it may contain (dashed), and who doesn't like an ingredient. Each ingredient that brings an allergen is marked on its line.
- **The week**: see [Week view](diary.md#week-view) and [Build My Week](diary.md#build-my-week).

It's a help, not a guarantee: it's worked out from ingredient names and pantry labels, so check the packaging when it matters.

## FDA-style Nutrition Facts box

The Nutrition column renders a full FDA-style label via `NutritionFactsBox.svelte`. Thirty-one nutriments in real label order, %DV column, sodium and salt auto-derived from each other (factor 400), servings and serving-size at the top. A per-user picker under Settings > Nutrition controls which rows the label shows inline; the rest still count for totals.

### Auto-recompute from Pantry

Tap **Recompute From Pantry** on the Nutrition column and every linked pantry row contributes its nutrition to the total, weighted by ingredient quantity. Volume-to-grams jumps use the density table and, cross-family (mass to volume), each pantry item's `g_per_cup` override. Ingredients without a linked pantry item or without a density get a skipped-item badge with an inline "Set N g/cup" chip so you can fill the gap in one tap.

## Sharing

Three separate sharing surfaces:

- **Per-user grants**: `POST /api/recipes/:id/shares` adds a grantee by user id. The recipe surfaces on their Shared tab.
- **Kitchens fan-out**: sharing a recipe with a Kitchen mints one grant per member, tagged `via_kitchen_id`. Removing a member (or leaving the Kitchen) revokes only grants tagged that way. See [Kitchens](kitchens.md).
- **Public share link**: `POST /api/recipes/:id/share` mints a random URL-safe token. The recipe is then readable without login at `/r/<token>` (older `/#/r/<token>` links still open). Bypasses the setup-required gate so a fresh install serving a shared link doesn't 503. `DELETE /api/recipes/:id/share` revokes. The page carries link-preview tags (the recipe's name, description, and photo) so chat apps show a preview, and asks search engines to skip it. Only these two routes set the token: sync sends it to devices but never takes it from them, so a removed link stays removed.

Card image: `GET /api/recipes/:id/card.png` returns a server-rendered Pinterest-style card for social sharing.

## Long-press and print

Long-press any recipe card for a menu: Open Recipe, Add to Favorites, Add to Cookbook, Create Share Link, Plan a Cook (the Diary's planner, with this recipe picked), Add to Shopping List, Duplicate, Share Card, Select Multiple, and Delete. The recipe view has a print stylesheet via `@media print` that strips chrome, sidebars, and interactive controls for a clean paper copy.

## Comments

Flat plus threaded comments per recipe. Anyone with read access can post; authors and admins can edit or delete. Reply notifications go out via the configured push provider when `notifRecipeComments` is on (default on).

## Related

- [Feature tour](features.md)
- [Pantry](pantry.md)
- [Cook Diary and Meal Planner](diary.md)
- [Import from URL, file, or photo](import.md)
- [Trace AI in CookTrace](trace.md)
