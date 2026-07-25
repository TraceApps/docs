# Kitchens

A Kitchen is a soft multi-user group: a household, a shared flat, a friend circle. Sharing a recipe or a cookbook with a Kitchen fans it out to every current member in one action, no per-user picking. Kitchens are a sharing macro, not a new ownership boundary. Recipes, pantry, diary, and shopping all stay per-user.

Kitchens only make sense with user management enabled. In single-user mode the UI still lets you create one so the feature isn't hidden, but there's nobody to invite.

## Creating a Kitchen

Under **Settings > Kitchens**, tap **Create Kitchen**. A name up to 80 characters is the only required field. Whoever creates the Kitchen becomes its owner and its first member. Owners can add and remove members and delete the Kitchen. Members can leave and can share their own recipes into the Kitchen.

## Adding members

From the Kitchen's member panel, type a username and hit **Add Member**. Lookup is case-insensitive against the users table. New members join with `auto_share = 0`, so they see other members' recipes if those other members have auto-share on, but their own library stays private until they turn it on themselves.

## Auto-Share Your Recipes

The per-member **Auto-Share Your Recipes** toggle is the flow that makes Kitchens click. Turning it on does two things:

1. **Backfills every recipe you already own** into the Kitchen: one `recipe_shares` row per (recipe, other member), tagged with `via_kitchen_id`.
2. **New members who join later automatically pick up your library** via the same fan-out, per the `POST /:id/members` hook.

Turning auto-share off does NOT revoke your existing grants. Kitchens are meant to be "still cook from what you already shared"; if you really want to nuke every prior grant, remove yourself from the Kitchen (or delete the recipes) instead.

Auto-share is independent per member and per Kitchen. You can be an auto-sharer in the "Home" Kitchen and a passive reader in the "Friends" Kitchen without conflict.

!!! note "Backfill is per-owner"
    Turning your auto-share toggle on backfills the recipes YOU own. Someone else's toggle handles their recipes. Nobody's toggle grants access to recipes they don't own.

## Sharing on demand

Even without auto-share on, you can share individual recipes or cookbooks with a Kitchen from their normal share dialogs:

- **Recipe**: `POST /api/kitchens/:id/share-recipe` with `{ recipe_id }`. Mints one `recipe_shares` row per member (except yourself). The requester has to own the recipe.
- **Cookbook**: `POST /api/kitchens/:id/share-cookbook` with `{ cookbook_id }`. Mints one `cookbook_shares` row per member, tagged `via_kitchen_id`. Recipes inside the cookbook that aren't independently shared render as [locked placeholders](cookbooks.md#locked-placeholders) to the recipient.

Both endpoints are idempotent thanks to `INSERT OR IGNORE` on the unique (recipe_id, grantee_id) / (cookbook_id, grantee_id) constraint. Calling twice doesn't duplicate rows.

## Leaving or being removed

`DELETE /api/kitchens/:id/members/:userId` handles both self-leave and owner-side removal. When a member leaves:

- Grants **into** them via this Kitchen (recipe and cookbook alike) get revoked. They stop seeing everyone else's shared library.
- Grants **from** them stay in place. Mom's meatloaf survives Mom leaving the family Kitchen; the remaining members keep cooking it.
- Manual per-user shares (`via_kitchen_id IS NULL`) are untouched either way.

Owners can't leave a Kitchen while still owning it (they'd orphan it); delete the Kitchen or transfer ownership first. Deleting a Kitchen removes the row and all memberships, but grants tagged with `via_kitchen_id` survive the delete (the column is `ON DELETE SET NULL`), so the shared history isn't punished by a housekeeping decision.

## The Shared tab, and blending

The Recipes page has a **Shared** tab that lists everything granted to you, with a Kitchen-name chip on rows that came through a Kitchen fan-out. If you'd rather see shared recipes inline with your own, toggle **Show Shared Recipes in My Main List** under Settings > Cooking.

The Cookbooks tab has an equivalent Shared section listing cookbooks granted to you (per-user or via Kitchen), with the same Kitchen-name chip.

## When Kitchens make sense

- Household of two or three cooks who want each other's recipes on tap without manually sharing every new one.
- A friend group swapping recipes casually; auto-share off, share on demand.
- A "test kitchen" for a couple where one person imports aggressively and the other picks favourites.

If you only ever share one or two recipes, per-user grants from the recipe's Share dialog are probably enough; skip the Kitchen.

## Related

- [Recipes](recipes.md)
- [Cookbooks](cookbooks.md)
