# NutriTrace federation (CookTrace side)

Wire CookTrace up to a NutriTrace instance and every recipe you build can auto-populate per-ingredient nutrition from foods you've already logged in NT. No more re-typing calories for the same tin of chickpeas.

For the high-level story and how the token model works across all three apps, see [Federation (cross-app links)](../integrations/federation.md).

## What CookTrace gains

- **NT-foods picker in the Pantry.** Search your NT foods library from CookTrace and one-tap import matched rows into your pantry, complete with brand, serving size, image, and nutrition.
- **Per-ingredient nutrition** on any recipe ingredient linked to a pantry row that came from NT. The FDA nutrition-facts box on the recipe page fills in automatically.
- **Barcode carry-over.** A pantry row with an NT-side barcode continues to scan correctly from CookTrace's own barcode scanner.

CookTrace does not push data back to NT via this integration. Cook-log fanout to NT's diary is scaffolded but not wired end-to-end (see the CookTrace roadmap).

## Configuring it

Mint the token in NutriTrace first: Settings > **API Tokens** > New Token, name it "CookTrace", tick **read:foods**, save. Copy the raw `nt_pat_...` value on the confirmation screen; NT only stores the hash after you close it.

Then in CookTrace, open Settings and expand **NutriTrace federation**:

1. **Instance URL**: your NT origin, e.g. `https://nutritrace.example.com` or `http://192.168.1.20:3000`. HTTP is allowed for LAN use; HTTPS is the sensible default for anything reachable off your LAN.
2. **Access Token**: paste the `nt_pat_...` value.
3. **Test Connection**: CookTrace calls NT `/api/v1/me` server-side. On success the UI shows "Connected as `<username>`". Errors call out the exact failure (bad URL, invalid token, missing scope).
4. **Enable Federation**: flip on. Save.

The token stays on CookTrace's own server. The WebView / browser never sees it, and NT calls are proxied through CookTrace's `/api/nt/*` routes.

## Which recipes get affected

Any recipe whose ingredient rows link to pantry items sourced from NT. Two paths get you there:

- **Bulk backfill**: Settings > **NutriTrace federation** > **Pull foods**. Search NT, pick the items you want, import into pantry in one shot. Existing pantry rows with the same name are skipped (not overwritten); soft-deleted ones are resurrected.
- **Per-ingredient**: while editing a recipe, use the pantry-link picker on an ingredient row and pick an NT-sourced pantry item.

Once linked, **Recompute from Pantry** on the recipe view sums each ingredient's contribution (using the built-in density table for volume-to-grams cross-conversion). Rows without a link surface a "Set N g/cup" affordance rather than silently dropping from the totals.

Barcode match wins over name match when both are present, so a scanned tin lines up with its NT record even if the recipe wrote the ingredient name slightly differently.

## Related

- [Federation (cross-app links)](../integrations/federation.md)
- [Federation API (v1)](../nutritrace/federation-api.md)
- [NutriTrace federation (LiftTrace side)](../lifttrace/nt-federation.md)
