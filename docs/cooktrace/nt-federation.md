# NutriTrace federation (CookTrace side)

Wire CookTrace up to a NutriTrace instance and every recipe you build can auto-populate per-ingredient nutrition from foods you've already logged in NT. No more re-typing calories for the same tin of chickpeas.

For the high-level story and how the token model works across all three apps, see [Federation (cross-app links)](../integrations/federation.md).

## What CookTrace gains

- **NT-foods picker in the Pantry.** Search your NT foods library from CookTrace and one-tap import matched rows into your pantry, complete with brand, serving size, image, and nutrition.
- **Per-ingredient nutrition** on any recipe ingredient linked to a pantry row that came from NT. The FDA nutrition-facts box on the recipe page fills in automatically.
- **Barcode carry-over.** A pantry row with an NT-side barcode continues to scan correctly from CookTrace's own barcode scanner.
- **Push a recipe to NutriTrace.** From any CookTrace recipe view, publish the recipe as a NutriTrace recipe entry (with per-ingredient nutrition and totals). See [Push a recipe to NutriTrace](#push-a-recipe-to-nutritrace) below.

Cook-log fanout to NT's diary is scaffolded but not wired end-to-end (see the CookTrace roadmap).

## Configuring it

Mint the token in NutriTrace first: Settings > **API Tokens** > New Token, name it "CookTrace", tick **read:foods**, save. Copy the raw `nt_pat_...` value on the confirmation screen; NT only stores the hash after you close it.

Then in CookTrace, open Settings and expand **NutriTrace federation**:

1. **Instance URL**: your NT origin, e.g. `https://nutritrace.example.com` or `http://192.168.1.20:3001`. HTTP is allowed for LAN use; HTTPS is the sensible default for anything reachable off your LAN.
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

## Push a recipe to NutriTrace

Open any recipe in CookTrace, scroll to the nutrition column, and hit **Push to NutriTrace**. The button only appears when NT federation is enabled and connected.

CookTrace will:

1. Run the client-side nutrition rollup (same code path as **Recompute from Pantry**) so any ingredients missing nutrition data or with a unit mismatch are collected into a "skipped" list.
2. Open a confirmation dialog showing the recipe name, the skipped-ingredient list (if any), and a warning that totals may be underestimated when ingredients are missing nutrition.
3. On confirm, POST the recipe (name, per-ingredient snapshots with any linked NT food id, rollup totals, serving info, image URL, and the skipped-ingredient warnings) to NT's `/api/v1/recipes` endpoint using the same access token that powers the food picker.
4. Record the returned NT recipe id on the CookTrace recipe row so the **next** push updates the same NT recipe rather than duplicating.

The token needs the `write:recipes` scope in addition to `read:foods`. Mint a fresh token with both scopes ticked, or edit your existing token to add the new scope, then paste the value in Settings.

### What NutriTrace shows

The pushed recipe appears in NT's Meals catalog with `is_recipe = 1`. Opening it in NT's Meal Editor surfaces:

- A **From CookTrace** badge with a link back to the original recipe on your CookTrace instance.
- If any ingredients were skipped during rollup, a warning banner listing them (up to five names) with the same "totals may be underestimated" note. This is stored in the recipe row's `import_warnings` and persists across NT restarts.

### Missing ingredient nutrition

CookTrace does not block the push when some ingredients lack nutrition data. The rationale: home cooks routinely enter a recipe with a spice or garnish they never bothered to log nutrition for; blocking the push over one row would be more friction than the caveat is worth. The banner on the NT side makes the "may be underestimated" caveat visible where it matters (at log time).

To tighten the totals after the fact, fill in the missing pantry item's nutrition on the CookTrace side and push again. The upsert semantics mean the NT recipe is updated in place, and once every ingredient has data the warning banner goes away.

### Re-pushing after edits

Pushing the same recipe twice does not duplicate it on NT. The identity key is `(NT user, source_app='cooktrace', source_external_id='recipe:<CT recipe id>')`, enforced by a partial unique index on the NT meals table. Edit the recipe on CookTrace, hit Push again, and the same NT row is updated with the new name, ingredients, and totals.

Deleting the CookTrace recipe does **not** delete the NT copy. NT treats the pushed recipe as a snapshot the user owns; deleting on the NT side is a separate action. If you want them tied, that would be a future enhancement worth an issue.

## Related

- [Federation (cross-app links)](../integrations/federation.md)
- [Federation API (v1)](../nutritrace/federation-api.md)
- [NutriTrace federation (LiftTrace side)](../lifttrace/nt-federation.md)
