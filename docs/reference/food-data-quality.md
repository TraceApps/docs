# Food data quality signals

CookTrace and NutriTrace both look up foods against **Open Food Facts** (OFF) and **USDA FoodData Central**. The two sources are enormous but uneven. To help you pick the trustworthy entry when several hits look similar, the food picker surfaces three visual signals on every result row: an **origin-country flag**, a **completeness dot**, and a **data-type badge**. All three are computed client-side from fields the source itself provides, so there is no extra API call and no risk of over-hitting rate limits.

The signals mean the same thing in both apps. LiftTrace does not touch food data, so it does not display any of them.

## Origin-country flag (OFF only)

Every Open Food Facts product record has an `origin` field naming the country the item is sold in. The picker renders that as a small country flag next to the product name. Useful because OFF is global: a search for "yogurt" pulls hits from France, Germany, the US, Brazil, and Japan, and you can pick the version that matches what's actually in your fridge in one glance.

If the product has no origin field, no flag is shown (which is itself a mild negative signal).

## Completeness dot (OFF only)

Each OFF record has a `completeness` percentage between 0 and 1 that measures how much of the standard field set is filled in (name, brand, ingredients, nutrition, allergens, packaging, images, and so on). The picker buckets it into a three-color dot:

- **Green** = complete or near-complete. Trustworthy nutrition, brand, and image data.
- **Yellow** = partial. Usually the basics are there but ingredients, allergens, or micronutrients might be missing.
- **Grey** = stub. Barcode and name only, or worse. Assume nothing beyond the name is accurate.

Rule of thumb: green dots are safe to log without checking; yellow dots deserve a glance before you commit; grey dots should only be used if you have no better option and you plan to edit the values.

## Data-type badge (USDA only)

USDA FoodData Central bundles five data types with meaningfully different trust levels. The picker renders whichever type applies as an inline badge:

- **Foundation Foods**: most trusted. Physically-analyzed samples from USDA's own labs, current data.
- **SR Legacy**: Standard Reference Legacy. Historical USDA analyzed foods. Excellent for generic ingredients (raw chicken breast, brown rice, olive oil) but frozen at the last release.
- **Survey (FNDDS)**: Food and Nutrient Database for Dietary Studies. Prepared and mixed dishes ("chicken enchiladas, homemade"). Modeled from Foundation and SR Legacy, so quality is one step removed.
- **Branded**: commercial packaged goods fed directly by manufacturers under FDA labeling rules. Reliable for macros; coverage is deep but not exhaustive.
- **Experimental**: provisional data. Rare, avoid unless you know what you're doing.

For raw ingredients (an apple, a chicken breast, rolled oats), prefer Foundation or SR Legacy. For brand-name packaged foods (a specific brand of yogurt or protein bar), prefer Branded. For homemade or restaurant-style dishes, Survey is often the only match.

## Tier filters

Both apps expose client-side filters that narrow results by any of the above signals:

- **OFF completeness tier**: `Full` / `Partial` / `Stub`
- **USDA data type**: multi-select across the five types above

Applying a filter is instant (client-side); it does not re-hit the API. Combine with the source-chip picker to answer a query like "only Foundation-grade raw ingredients from USDA" in one click.

## Where the signals appear

- **CookTrace** displays them in **Pantry → External lookup** (Open Food Facts + USDA search) and in the pantry-link picker inside the recipe editor. See [CookTrace → Pantry](../cooktrace/pantry.md).
- **NutriTrace** displays them in the **Foods** picker across every meal-add flow, and in the Foods library search. See [NutriTrace → Foods, meals & recipes](../nutritrace/foods.md).

The underlying source data is identical between the two apps; the signals render the same way. If you find a bad-quality entry in one app, it will be bad in the other; correcting the OFF or USDA record itself fixes both.

## Contributing back to Open Food Facts

If you scan a barcode that OFF doesn't have (grey dot everywhere, or no results), both apps offer a **contribute** action: the barcode is prefilled in the food editor, you fill in the fields you know, and the entry uploads to OFF under your account. Doing this once takes about 60 seconds and makes the record available to every OFF consumer forever, including future you.

NutriTrace additionally has a **Share to Open Food Facts** button that promotes a food you added locally into the OFF community database, with the same round-trip.

## Related

- [CookTrace → Pantry](../cooktrace/pantry.md)
- [NutriTrace → Foods, meals & recipes](../nutritrace/foods.md)
- [NutriTrace → Open Food Facts](../nutritrace/off.md) covers the local OFF DuckDB mirror.
- [NutriTrace → USDA and Mealie integration](../nutritrace/usda-mealie.md) covers USDA API setup.
