# NutriTrace federation (CookTrace side)

Two flows live between CookTrace and NutriTrace, one in each direction.

- **CookTrace pulls foods from NutriTrace** so recipe ingredients can be auto-populated with the same nutrition you already log in NT. Minted on NT with `read:foods`, pasted into CookTrace.
- **NutriTrace pulls recipes from CookTrace** so recipes you built here show up as a source chip in NT's Foods search and can be imported into NT's recipes catalog. Minted on CookTrace with `read:recipes`, pasted into NutriTrace.

Both are opt-in, per-user, and independent of each other. Configure the one you want, or both.

For the high-level story and how the token model works across all three apps, see [Federation (cross-app links)](../integrations/federation.md).

## Pull foods from NutriTrace (CT is the client)

### What CookTrace gains

- **NT-foods picker in the Pantry.** Search your NT foods library from CookTrace and one-tap import matched rows into your pantry, complete with brand, serving size, image, and nutrition.
- **Per-ingredient nutrition** on any recipe ingredient linked to a pantry row that came from NT. The FDA nutrition-facts box on the recipe page fills in automatically.
- **Barcode carry-over.** A pantry row with an NT-side barcode continues to scan correctly from CookTrace's own barcode scanner.

Cook-log fanout to NT's diary is scaffolded but not wired end-to-end (see the CookTrace roadmap).

### Configuring it

Mint the token in NutriTrace first: Settings, **API Tokens**, New Token, name it "CookTrace", tick **read:foods**, save. Copy the raw `nt_pat_...` value on the confirmation screen; NT only stores the hash after you close it.

Then in CookTrace, open Settings and expand **NutriTrace federation**:

1. **Instance URL**: your NT origin, e.g. `https://nutritrace.example.com` or `http://192.168.1.20:3001`. HTTP is allowed for LAN use; HTTPS is the sensible default for anything reachable off your LAN.
2. **Access Token**: paste the `nt_pat_...` value.
3. **Test Connection**: CookTrace calls NT `/api/v1/me` server-side. On success the UI shows "Connected as `<username>`". Errors call out the exact failure (bad URL, invalid token, missing scope).
4. **Enable Federation**: flip on. Save.

The token stays on CookTrace's own server. The WebView / browser never sees it, and NT calls are proxied through CookTrace's `/api/nt/*` routes.

### Which recipes get affected

Any recipe whose ingredient rows link to pantry items sourced from NT. Two paths get you there:

- **Bulk backfill**: Settings, **NutriTrace federation**, **Pull foods**. Search NT, pick the items you want, import into pantry in one shot. Existing pantry rows with the same name are skipped (not overwritten); soft-deleted ones are resurrected.
- **Per-ingredient**: while editing a recipe, use the pantry-link picker on an ingredient row and pick an NT-sourced pantry item.

Once linked, **Recompute from Pantry** on the recipe view sums each ingredient's contribution (using the built-in density table for volume-to-grams cross-conversion). Rows without a link surface a "Set N g/cup" affordance rather than silently dropping from the totals.

Barcode match wins over name match when both are present, so a scanned tin lines up with its NT record even if the recipe wrote the ingredient name slightly differently.

## Pull a CookTrace recipe into NutriTrace (CT is the server)

The other direction: your CookTrace recipes show up as an on-demand source in NutriTrace's Foods search, so you can import them into NT's recipes catalog without leaving NT. Mirrors NT's Mealie integration in shape and posture, but sourced from your own CookTrace instance.

### Mint a token in CookTrace

CookTrace ships the same personal-access-token model as NutriTrace. Open Settings, expand **API Tokens**, click **New Token**. Name it something like "NutriTrace recipe pull". Tick **read:recipes** (the only scope this flow needs; leaves your MCP scopes out of it). Save. The raw value is shown once, formatted `ct_pat_<43 chars>`. Copy it now, because CT only stores the hash after the confirmation screen closes.

### Configure NutriTrace

In NutriTrace, open **Settings, Connected Services, CookTrace**:

1. **Base URL**: your CookTrace origin, e.g. `https://cooktrace.example.com`.
2. **API Token**: paste the `ct_pat_...` value.
3. **Enable**: flip on. On save NT hits `/api/v1/me` on your CookTrace server through its own proxy (`POST /api/cooktrace/proxy`) and reports "Connected as `<username>`" or a specific failure (URL wrong, token invalid, or `read:recipes` scope missing).

The token stays on NT's own server. NT's browser and mobile app never see it; the proxy forwards each request server-side.

### How the pull works in NT

Open the Foods screen, tap the **Recipes** tab, hit the source chip row. A **CookTrace** chip appears next to Local (and next to From Others, when applicable). Tap it and type: NT proxies each keystroke to `GET /api/v1/recipes?q=...` on your CookTrace instance and lists the matches by name.

Pick a recipe: NT fetches `GET /api/v1/recipes/<id>`, opens NT's Recipe editor pre-filled with:

- **Name, servings, image** from the CookTrace recipe row.
- **Ingredients**, one row per CT ingredient, with the pantry-link brand and name where available, plus the per-ingredient nutrition snapshot CT stores when the ingredient is linked to a pantry item.
- **Rollup totals** (whatever CookTrace has stored from its most recent Recompute). If the recipe was never Recomputed, totals may be empty; NT lets you run its own Recompute after import to fill them in from the NT foods library.

Save the recipe and it lands in NT's Recipes catalog with a **From CookTrace** badge and a back-link to the original recipe on CookTrace. If any ingredients came in without nutrition, NT surfaces them as ordinary loose items ready for you to link or fill in on the NT side.

### Re-pulling after edits

Every save on the NT side stamps `(source_app='cooktrace', source_external_id='recipe:<id>')` on the meals row. Re-pulling the same recipe and saving again updates the same row rather than duplicating (partial unique index on `meals(user_id, source_app, source_external_id)`), so edits on CookTrace flow through to NT with one pick + save.

### Auto-refresh prompt

When you open a CookTrace-imported recipe in NT's Meal Editor, NT quietly checks the CookTrace source in the background. If CookTrace's `updated_at` is newer than your local NT copy, a **CookTrace has newer content** banner appears above the recipe with **Refresh** and **Dismiss** buttons. Tapping **Refresh** re-fetches the recipe, remaps ingredients + totals, and applies them to the current row so the next save persists the update. Never overwrites without your consent, silent on any failure (offline, missing token, etc.), and never fires when CookTrace federation is disabled.

### Source-deleted indicator

If that same background probe comes back with a hard **404** (the CookTrace recipe was deleted, or the token lost read access to it), NT shows a warning-tinted **CookTrace source no longer available** banner in place of the refresh prompt, with two actions:

- **Unlink**: strips the CookTrace provenance from the NT copy but keeps the recipe. The row becomes an ordinary NT-authored recipe with no further sync attempts.
- **Delete**: removes the NT copy entirely. Diary entries that already referenced it stay logged with the nutrition they captured at log time.

A network failure or other non-404 error does **not** trigger the banner (offline users would otherwise see it constantly); only an unambiguous "gone" from CookTrace does.

### Bulk import all recipes

Instead of pulling recipes one at a time from the source-chip picker, you can also grab everything CookTrace has in one shot. In NutriTrace open **Settings, Connected Services, CookTrace** and use the **Import All Recipes** action once the connection is verified. NT pages through every recipe on the CookTrace side, upserts each one via the same `(source_app, source_external_id)` dedup key the single-recipe picker uses, and reports imported / updated / skipped counts. Safe to re-run: subsequent invocations update existing imports in place rather than duplicating.

## Pull CookTrace pantry items into NutriTrace foods

The reverse of the "CookTrace pulls foods from NutriTrace" flow above: NT can also pull the pantry items you keep on CookTrace into its own foods library. Useful for onboarding NutriTrace when you have been living in CookTrace's pantry for a while.

### Mint a token with `read:pantry`

The scope is separate from `read:recipes` on purpose (you may want to share only one). Open CookTrace **Settings, API Tokens, New Token**, tick **read:pantry** (add **read:recipes** too if you also want the recipe-pull flow above), save, copy the `ct_pat_...` value.

### Run the import in NutriTrace

In NutriTrace **Settings, Connected Services, CookTrace**, once the connection is verified, use **Import Pantry Items**. NT pulls every leaf pantry row from CookTrace and upserts it into your foods library via the same `(source_app='cooktrace', source_external_id='pantry:<id>')` dedup key the recipe flow uses.

Leaf-only rule: **generic parents with variants are skipped**. Their variants carry the real nutrition; a top-level "Flour" placeholder next to "Flour, Bread" and "Flour, All-Purpose" in your foods library would be misleading and unusable. Standalone pantry items and individual variants both come across; variants get a "Parent, Child" display name (e.g. "Flour, Bread") so they read cleanly in NT's foods list.

Nutrition per row uses the same resolver the recipe pull uses, so a variant whose own row has no nutrition inherits from the parent's designated variant if one is set. Rows that resolve to nothing still come across (as empty-nutrition placeholders); NT lets you fill them in on the NT side.

## How images come across

Both flows carry an `img_url` for recipes and pantry items. On import, NutriTrace's server downloads the image and self-hosts it under `/uploads/`, so from that point on every client viewing the recipe or pantry row loads the picture from NutriTrace's own origin (no runtime dependency on the CookTrace host).

For the download step to succeed, **the NutriTrace server** must be able to reach the CookTrace origin. Two setups behave differently:

- **NutriTrace server and CookTrace on the same LAN, or both on the same public origin.** The download works, the image lands in `/uploads/`, and every client sees the thumbnail. NutriTrace trusts the CookTrace base URL you saved in Settings so a private-IP LAN address is not refused by NutriTrace's SSRF guard.
- **NutriTrace on a public origin and CookTrace on a LAN address the NT server cannot reach** (or the other way around). The NutriTrace server literally has no network path to the CookTrace host, so the download fails. The recipe or pantry row still imports fine (name, ingredients, nutrition, everything else); only the thumbnail is missing. Edit the imported row and re-upload the photo locally on the NutriTrace side if you need it.

Nothing on the client (browser or Android WebView) has to reach CookTrace after import; whether the thumbnail shows up depends only on the NT server's reachability at import time.

Deleting the CookTrace recipe does **not** automatically delete the NT copy. NT treats the imported recipe as a snapshot the user owns; deletion is triggered explicitly via the source-deleted indicator's **Delete** action or by opening the recipe in NT and using the standard delete affordance.

## Related

- [Federation (cross-app links)](../integrations/federation.md)
- [Federation API (v1) on NutriTrace](../nutritrace/federation-api.md)
- [NutriTrace federation (LiftTrace side)](../lifttrace/nt-federation.md)
