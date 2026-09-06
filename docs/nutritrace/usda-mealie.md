# USDA and Mealie integration

Two optional per-user sources that live alongside Open Food Facts (OFF) in the Foods picker. **USDA FoodData Central** gives you the US government's own nutrition database (better than OFF for US foods, narrower coverage). **Mealie** pulls your self-hosted recipes into the picker so you can log them the same way you log any other meal.

Both are free. Both are configured per-user under **Settings → Connected Services**. Neither requires anything at the container level unless you want to env-lock the setting.

## USDA FoodData Central

FoodData Central is the USDA's public nutrition database: Foundation Foods, SR Legacy, Survey (FNDDS), Branded Foods, and Experimental. Coverage is deep for US grocery products and generic ingredients; thin for anything outside the US.

### Get an API key

1. Open [api.data.gov/signup](https://api.data.gov/signup/).
2. Enter your name and email. No account, no payment.
3. The key arrives by email in seconds. It is a plain string, no expiration under normal use.
4. Rate limit is 1000 requests per hour per key. Well beyond what a household instance will ever hit.

### Configure NutriTrace

**Settings → Connected Services → USDA FoodData Central**:

1. Toggle **Enable** (`usdaEnabled`).
2. Paste the API key (`usdaApiKey`).
3. Save.

A **USDA** source chip appears at the top of the Foods picker. Tap it to scope search to USDA, or long-press to combine with other sources (Local / OFF / Mealie / From Others). Each result row carries a **data-type badge** (Foundation / SR Legacy / Survey / Branded / Experimental) so you can prefer higher-quality entries. A **USDA data type** tier filter narrows the list further; entirely client-side, no extra API calls. See [Food data quality signals](../reference/food-data-quality.md) for what each data type actually means and when to prefer which.

### When USDA beats OFF

- **Generic ingredients** ("chicken breast, raw", "oatmeal, dry", "olive oil"): USDA's Foundation and SR Legacy entries are peer-reviewed with full micronutrient coverage. OFF's generics tend to be thinner.
- **US brand-name packaged foods**: Branded Foods in USDA is fed directly by manufacturers under FDA labeling. Reliable macros, though brand coverage is not exhaustive.
- **Micronutrients**: USDA carries a much wider set of tracked nutrients out of the box.

### When OFF beats USDA

- **Non-US products**: OFF is global; USDA is US-only.
- **Barcodes**: OFF is barcode-native; USDA's Branded Foods have GTINs but not with the same UI-first design.
- **Community contributions**: OFF is a wiki, so obscure or seasonal items land there first.

Run both. The chip filter lets you pick per-search.

## Mealie

[Mealie](https://mealie.io/) is a self-hosted recipe manager (Vue/FastAPI). If you keep your recipes there, NutriTrace can list them in the Foods picker and log them like any other meal, with the ingredients auto-mapped to NutriTrace foods where possible.

### Prerequisite: a running Mealie instance

Mealie needs to be reachable from your NutriTrace server (not from the browser). If both are in Docker on the same host, use the internal container name (`http://mealie:9000`); if Mealie is elsewhere, use its full HTTPS URL. Grab a **personal API token** from Mealie's user settings.

### Configure NutriTrace

**Settings → Connected Services → Mealie**:

1. Toggle **Enable** (`mealieEnabled`).
2. **Base URL** (`mealieBaseUrl`): the URL Mealie is served at (`https://mealie.example.com` or `http://mealie:9000`).
3. **API token** (`mealieApiToken`): the personal token from Mealie.
4. Save.

The NutriTrace server proxies Mealie requests via `POST /api/mealie/proxy`, so the token never touches the browser. A **Mealie** source chip appears in the Foods picker.

### How Mealie recipes are imported

When you pick a Mealie recipe from the picker, NutriTrace imports it as a NutriTrace recipe (same shape as any recipe you would build in the Recipe editor):

- **Name, description, servings, image**: copied verbatim.
- **Ingredients**: each Mealie ingredient is matched against your Local foods first, then OFF, then USDA. Matches with high confidence are auto-linked; the rest surface in the editor with a **needs mapping** badge for you to resolve.
- **Instructions**: preserved as recipe notes.

Once imported, the recipe lives in your NutriTrace Recipes catalog and can be logged, split, shared, and favorited exactly like any other recipe. It is not kept in sync with Mealie; if you change the source recipe, re-import to pick up the changes.

## CookTrace

[CookTrace](https://cooktrace-media.github.io/cooktrace/) is the recipes sibling in the TraceApps family. If you keep recipes there, NutriTrace can pull them into the Foods picker's **Recipes** tab as a source chip, mirroring the Mealie flow: pick, open in NT's Recipe editor pre-filled, save into your NT recipes catalog.

### Prerequisite: a running CookTrace instance with a token

CookTrace needs to be reachable from your NutriTrace server (not from the browser). Mint a **personal API token** in CookTrace: Settings, API Tokens, New Token, tick `read:recipes`, save. Copy the raw `ct_pat_...` value on the confirmation screen; CT only stores the hash after you close it.

### Configure NutriTrace

**Settings, Connected Services, CookTrace**:

1. Toggle **Enable** (`cooktraceEnabled`).
2. **Base URL** (`cooktraceBaseUrl`): the URL CookTrace is served at (`https://cooktrace.example.com` or `http://cooktrace:3003`).
3. **API Token** (`cooktraceApiToken`): the `ct_pat_...` value.
4. Save. NutriTrace hits CookTrace's `/api/v1/me` server-side and reports the CookTrace username on success, or a specific failure (URL wrong, token invalid, `read:recipes` scope missing).

The NutriTrace server proxies CookTrace requests via `POST /api/cooktrace/proxy`, so the token never touches the browser. A **CookTrace** source chip appears on the Foods screen, Recipes tab.

### How CookTrace recipes are imported

When you pick a CookTrace recipe from the picker, NutriTrace opens the Recipe editor with:

- **Name, servings, image** copied from CT.
- **Ingredients** as one editor row per CT ingredient, with the per-ingredient nutrition snapshot CookTrace stores when the ingredient is linked to a pantry item. Ingredients without a pantry link land as loose rows for you to link on the NT side.
- **Rollup totals** from CT's most recent Recompute (empty if the recipe was never Recomputed; run NT's own Recompute after import to fill from the NT foods library).

Save the recipe and it lands in NT's Recipes catalog with a **From CookTrace** badge and a back-link to the original recipe on CookTrace. Re-pulling and saving again updates the same row rather than duplicating, so edits on CookTrace flow through with one pick + save.

See [Pull a CookTrace recipe into NutriTrace](../cooktrace/nt-federation.md#pull-a-cooktrace-recipe-into-nutritrace) for the CookTrace-side setup and the full field-by-field wire contract.

## USER_PREFS vs DEVICE_PREFS

`usdaEnabled`, `usdaApiKey`, `mealieEnabled`, `mealieBaseUrl`, `mealieApiToken`, `cooktraceEnabled`, `cooktraceBaseUrl`, and `cooktraceApiToken` are all `USER_PREFS`: they sync with the user across devices. The Mealie and CookTrace API tokens stay on the server side of every federation request (the browser posts to `/api/mealie/proxy` and `/api/cooktrace/proxy`; only the server sees the outbound Bearer header).

## Related

- [Foods, meals and recipes](foods.md)
- [Open Food Facts](off.md)
- [Barcode and Scan Label](scan.md)
