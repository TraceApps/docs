# Cookbooks

Cookbooks are named collections of recipes. Use them for anything: "Weeknight", "Holiday", "Mom's recipes", "Currently trying". A recipe can live in as many cookbooks as you want; the join is a link table, not an ownership move.

## Regular cookbooks

Create one under **Manage > Cookbooks** (or from the Cookbooks tab on the Recipes page). Fields:

- **Name** (required).
- **Description** (optional).
- **Cover image** (optional URL or upload).

Add recipes from the cookbook detail page (**+ Add Recipes**) or from any recipe view's cookbook picker. Inside the cookbook, drag any card to reorder or use the arrow buttons for one-notch nudges; order persists via `PUT /api/cookbooks/:id/recipes/order`.

The cookbook list itself is also drag-to-reorder (or use **Move Up / Move Down** buttons); the global order writes via `PUT /api/cookbooks/order`.

## Move and Copy inside a cookbook

Each recipe card on a cookbook detail page has **Move** and **Copy** buttons for shuffling recipes between cookbooks without going back to the recipe view. Move rewrites the link's cookbook, Copy adds a second link so the recipe lives in both.

## Smart cookbooks

Toggle **Smart** at create time and the cookbook stops being a link table. It becomes a saved filter that re-evaluates every time you open it. Filter fields:

- **Category**: a single recipe category.
- **Tags**: AND-matched, so a recipe needs every listed tag.
- **Favorites only**.
- **Minimum rating** (1 to 5).
- **Maximum total minutes** (prep plus cook).

The evaluator lives in `_evalSmartFilter` in `server/routes/cookbooks.js`. Smart cookbooks always target the owner's recipes, so a viewer with access sees exactly what the owner sees (with the same locked-placeholder rules as manual cookbooks below). Toggling Smart off blanks the filter, and any manual link rows resume control of contents.

## Sharing

Two share surfaces, both mirroring how [Recipes](recipes.md) share:

- **Per-user grants**: `POST /api/cookbooks/:id/shares` with `{ user_ids: [...] }`. The cookbook shows up on the recipient's Cookbooks > Shared tab with a "Shared by X" chip. Grants are silently deduplicated; sharing a second time with the same user is a no-op.
- **Kitchen fan-out**: `POST /api/kitchens/:id/share-cookbook`. Grants one row per current member, tagged with `via_kitchen_id`. Leaving or being removed from the Kitchen revokes only these tagged grants. See [Kitchens](kitchens.md).

### Locked placeholders

Cookbook access is not transitive to the recipes inside it. If someone shares a cookbook with you that includes a recipe they haven't also granted you access to (directly or via a Kitchen), that recipe renders as a locked placeholder: only `id`, `name`, and `link_order` come through, everything else scrubbed. This is intentional per the audit and the `hydratedRecipes` gate in `server/routes/cookbooks.js`; a shared cookbook can't be used to launder access to recipes that were meant to stay private.

To grant read on a specific recipe inside a shared cookbook, share the recipe directly (per-user or via Kitchen). The locked placeholder then hydrates into a full card on the next load.

## Deletion

`DELETE /api/cookbooks/:id` soft-deletes the cookbook and hard-deletes its `recipe_cookbook_links` rows so a future cookbook with a recycled id can't inherit stale recipes. Recipes themselves are never touched; only the collection goes away.

## Related

- [Recipes](recipes.md)
- [Kitchens](kitchens.md)
