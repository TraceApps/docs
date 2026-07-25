# Migrate from Mealie

Mealie's export format is well-defined and CookTrace's bulk import understands it out of the box. The whole move takes minutes for a small library and a coffee break for a big one.

## What carries over

- **Recipes**: name, description, hero image, prep/cook times, ingredients, steps, notes, source URL, rating, tags, and categories.
- **Categories**: each Mealie category becomes a CookTrace Recipe Category on your account.
- **Images**: the hero image and any embedded step images.
- **Tags**: Mealie tags land on the recipe as CookTrace tags.

What doesn't carry over:

- Mealie users and permissions (CookTrace has its own user model).
- Mealie meal plans (CookTrace's Meal Planner is per-user, no cross-user plans).
- Comments (Mealie stores them, CookTrace's comment thread starts fresh on the imported recipe).

## Export from Mealie

In Mealie:

1. Log in as an admin.
2. Open **Settings > Maintenance > Backups**.
3. Click **Create Backup**. Mealie builds a `.zip` containing the recipe JSON tree and images.
4. Download the zip to your workstation.

The Mealie backup format is the whole household's recipes plus assets. CookTrace's importer walks the zip, finds every recipe JSON file, and pairs it with the image it references.

## Import into CookTrace

In CookTrace:

1. Open **Settings > Import from Another App**.
2. Click **Bulk Import**.
3. Drop the Mealie backup zip. Upload progress shows.
4. The **scan** step runs on the server, unpacking the zip into `.import-cache/<uuid>/` and returning a list of every recipe found, each with a thumbnail.
5. Review the list. Everything is selected by default. Uncheck anything you don't want.
6. Click **Commit** to import the selected recipes.

The commit response tells you how many landed, how many were skipped as duplicates, and how many failed to parse (if any).

!!! tip "Big library?"
    The default cap is `IMPORT_ZIP_MAX_MB=512` MB, which is enough for most Mealie backups. Bump it in your env if you overflow, then restart the container. See [env vars](env-vars.md).

## Image dedup

CookTrace stores images under `/data/uploads/`. The importer copies hero images from the Mealie archive into `uploads/`, hashes them, and reuses an existing file when the hash already exists in your library. Step images work the same way. Re-running the import (with `dedup: 'force'` if you want a fresh copy) doesn't blow up disk usage.

## Category carry-over

Mealie categories map to CookTrace **Recipe Categories** on your user account. If you already have categories with matching names, the importer reuses them; otherwise it creates them. Colours are assigned from the default palette, tweak them later under **Manage > Recipe Categories**.

Tags come across as CookTrace tags (per-user catalog). Recipe types and course info that Mealie split across multiple fields collapse into tags when there's no direct category equivalent.

## Duplicates

The default bulk-import policy is `dedup: 'skip'`. A recipe is treated as a duplicate when either:

- The name matches an existing recipe on your account (case-insensitive), or
- The `source_url` matches an existing recipe on your account.

Skipped duplicates are reported in the commit result so you can spot-check. If you want to reimport intentionally (say, Mealie's copy has been updated), use the individual **Import** path with `dedup: 'force'`, or delete the CookTrace copy first.

## After importing

- Open **Manage > Recipe Categories** and tidy names and colours if you had inconsistent capitalisation in Mealie.
- If you use the Pantry, enable **Settings > Cooking > Auto-Add Ingredients to Pantry** and re-save a couple of recipes to backfill pantry entries. Or bulk-add the ingredients from **Settings > NutriTrace** if you have a NutriTrace instance handy.
- Trace can help too: ask "review my new imports and add missing tags for cuisine" and it'll walk the batch.

## Related

- [Import from URL, file, or photo](import.md)
- [Env vars](env-vars.md)
- [Backups](../self-hosting/backups.md)
