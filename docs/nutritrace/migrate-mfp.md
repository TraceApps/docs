# Migrate from MyFitnessPal

Two supported paths, with different tradeoffs. Pick one based on how much of your MFP history you actually want back.

## The short answer

- **Just your custom foods, plus per-meal daily calorie totals**: use MFP's official export. Import runs through **Settings → Import & Export → Import from another app → MyFitnessPal** with preview and per-date conflict policy.
- **Per-food diary rows too (what MFP's export leaves out)**: use the community scraper at [github.com/nomad64/mfp-to-nutritrace](https://github.com/nomad64/mfp-to-nutritrace), which uses your browser session cookies to pull the per-food rows MFP's own export omits.

Both paths land data in the same NutriTrace tables (`diary`, `foods`) via the same import endpoints, so mixing them is fine: run the official export first for the base coverage, then the scraper for any date range you want re-imported at higher fidelity.

## Path A: MFP's official export

MFP's export lives at [myfitnesspal.com/exercise/diary](https://www.myfitnesspal.com/) under the account settings (the exact path shifts as MFP redesigns; search for "download my data" in your account). The export includes:

- **My Foods**: the custom foods you have added over the years. Full macros.
- **Diary**: per-meal totals per day. So "Lunch: 640 kcal, 22g protein, 40g carbs, 30g fat" but **not** the individual foods that added up to that.

That last bit is the catch. If you value having your historical per-food rows in NutriTrace ("I ate a Chipotle burrito on the 14th"), MFP's export does not give you that; go to Path B below for those.

### Import My Foods

**Settings → Import & Export → Bulk Import** takes the JSON or CSV format documented inline in the panel. MFP's exported foods CSV maps directly with the built-in MFP profile: pick **MyFitnessPal (My Foods)** from the source picker, upload the file, and click **Preview**. The preview lists every food that would be inserted, marks any duplicates against your existing catalog, and lets you pick whether to skip or overwrite dupes. Confirm to commit.

### Import Diary days

Same panel, **MyFitnessPal (Diary)** source. Upload the diary export, preview, pick a **per-date conflict policy**:

- **Skip**: leave any date that already has diary data untouched.
- **Merge**: add MFP rows alongside anything that's already there.
- **Overwrite**: replace whatever is on that date with the MFP rows.

Because MFP's diary export is per-meal totals only, the imported days show a single row per meal with the meal-level macros, not individual food rows. This is enough for the Statistics page and Adaptive TDEE math (both operate on daily totals), but any per-food question you ask retroactively will hit that "sum only" limitation.

## Path B: nomad64's community scraper

[github.com/nomad64/mfp-to-nutritrace](https://github.com/nomad64/mfp-to-nutritrace) is an unaffiliated community tool that scrapes MFP diary pages via your browser session cookies and produces files in NutriTrace's bulk-import shape. Two scripts:

- **My Foods exporter**: dumps your custom foods to NutriTrace's bulk-import JSON. Similar coverage to MFP's official export but with a few edge cases handled better (units, notes).
- **Per-food diary scraper**: walks the date range you specify and pulls the individual food rows MFP's official export omits. Output is JSON in NutriTrace's per-food diary import shape.

Because the scraper uses your session cookies, treat it like any other cookie-holding tool: run it locally, don't post its output publicly, and rotate your MFP password when done if you're paranoid. It's a Python script; the README on nomad64's repo covers install and usage.

**Import the output** via **Settings → Import & Export → Import from another app → MyFitnessPal** (same panel as Path A but this time with the scraper's JSON, not MFP's CSV). Preview shows per-food rows on their real meal slots; commit with the conflict policy of your choice.

## Preview + conflict policy (both paths)

Every commit goes through `POST /api/nutrition-import/preview` first. The preview endpoint returns the exact set of foods and diary rows that would be inserted, per-date grouping, dupe-detection against your existing catalog by name + brand, and any parse warnings. Only after you confirm does the commit endpoint (`POST /api/nutrition-import/commit`) actually write.

The per-date conflict policy applies at commit time and can be different for different date ranges within the same import (the UI groups dates so you can, say, skip everything before 2024 and overwrite everything after).

## After you import

- **Diary** shows the imported days immediately. Use the date picker to walk back through them.
- **Statistics** picks up historical data on the next chart render; nothing to refresh manually.
- **Adaptive TDEE** benefits from the extra days if you also have weight data covering them. If you don't, log historical weights via **Diary → Body Stats → Add** on the relevant dates; body stats accept backdated entries.
- **Foods you imported** land in your Local catalog with a `source: mfp` tag so you can filter on origin later.

Bulk-share to family members on the same instance happens the same way as any other food (**Settings → Sharing → Bulk share**). Nothing about the MFP origin restricts sharing.

## Related

- [Migrate from Lose It / Cronometer / Waistline](migrate-other.md)
- [Foods, meals & recipes](foods.md)
- [Diary & meal logging](diary.md)
