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

Deleting the CookTrace recipe does **not** delete the NT copy. NT treats the imported recipe as a snapshot the user owns; deleting on the NT side is a separate action.

## Related

- [Federation (cross-app links)](../integrations/federation.md)
- [Federation API (v1) on NutriTrace](../nutritrace/federation-api.md)
- [NutriTrace federation (LiftTrace side)](../lifttrace/nt-federation.md)
