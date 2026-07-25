# Migrate from Tandoor

Tandoor exports recipes as JSON. CookTrace's bulk-import flow accepts either single-file JSON drops (one recipe) or the multi-recipe zip Tandoor produces from its bulk export UI.

## Export from Tandoor

Two options depending on library size.

### Bulk export UI

The easier path when you want everything:

1. In Tandoor, open **Settings > Import / Export**.
2. Under Export, pick **JSON** as the format and **All recipes** as the scope.
3. Download the resulting zip. Each recipe is a `.json` file inside it.

### Per-recipe API

Handy for scripting or partial exports. Tandoor exposes recipes via its REST API:

```bash
curl -H "Authorization: Bearer <token>" \
  "https://tandoor.example.com/api/recipe/?export=json&page_size=200" \
  > tandoor-recipes.json
```

Or one recipe at a time:

```bash
curl -H "Authorization: Bearer <token>" \
  "https://tandoor.example.com/api/recipe/<id>/?export=json" \
  > recipe.json
```

## Import into CookTrace

Under **Settings > Import from Another App**:

- **Single JSON**: drop the file into the File / Paste dialog. The detector recognises Tandoor's shape (steps carrying nested ingredients, plus a top-level name and `working_time`) and imports one recipe.
- **Bulk zip**: use the **Bulk Import** card. The server unpacks the zip into `.import-cache/<uuid>/`, walks every `.json` at any depth, detects each as Tandoor, and returns a picker with thumbnails. Confirm and commit.

The commit response reports how many landed, how many were skipped as duplicates (case-insensitive name match or exact `source_url` match), and how many failed to parse. See [dedup policy](import.md#dedup-policy) for the `force` option that bypasses the skip.

## What carries over

Per `_importTandoor` in `server/lib/recipe-importers.js`:

- **Name**, **description**, **hero image** (Tandoor's `image` URL).
- **Servings** and yield text.
- **Prep** time from Tandoor's `working_time`, **cook** time from `waiting_time`.
- **Steps**: each Tandoor step's `instruction` becomes one CookTrace step. Named step groups (Tandoor lets you name each step) preserve the naming on the ingredient side.
- **Ingredients**: each Tandoor step's `ingredients[]` becomes an ingredient group on the CookTrace recipe. `amount` maps to qty, `unit.name` to unit, `food.name` to the ingredient text, `note` to the note field. If there's only one unnamed group, the group name flattens away and you get a single ungrouped list.
- **Keywords** land on CookTrace **tags** (up to 12).
- **Source URL** (`source_url`).

## What doesn't

- **Category**: Tandoor doesn't have first-class categories. It uses keywords for both tags AND categorisation. Since those keywords already flow into the tags array on import, promoting one to the category slot would be guesswork. Category is left blank on purpose; assign one in CookTrace after the fact if you want the recipe filed under a category too.
- **Meal plans** and **shopping lists**: separate Tandoor exports, not ingested. Rebuild in [Cook Diary](diary.md) and let [Shop This Plan](shopping.md#shop-this-plan) generate the list.
- **Nutrition**: passed through as-is when present, but Tandoor's nutrition JSON is loosely typed. **Recompute From Pantry** on the recipe view is usually the cleaner path once your imported ingredients are linked to pantry rows.

## Unit conversion mismatches

Tandoor's units are free-form strings (whatever names you created in Tandoor). CookTrace's [37-unit catalog](recipes.md#inline-unit-converter) is more opinionated. Common cooking units (cup, tbsp, tsp, g, kg, ml, l, oz, lb) round-trip cleanly and the inline converter handles them without help.

Anything Tandoor-specific ("Prise", "Handful", or your own custom units) lands as a raw unit string. The inline converter won't know what to do with it, and Recompute From Pantry will skip that row with a "no density" badge until you either edit the unit into one the catalog knows, or add a matching custom unit under **Manage > Units**.

If you're doing a large migration, turn on `LOG_LEVEL=debug` before running the bulk commit and check Settings > Diagnostics > View Diagnostic Logs afterwards to spot any unit strings that fell through.

## Size cap

`IMPORT_ZIP_MAX_MB` defaults to **512** MB. Bump it in your environment if your archive is bigger. See [env vars](env-vars.md).

## Related

- [URL, file, and photo import](import.md)
- [Migrate from Paprika](migrate-paprika.md)
- [Migrate from Mealie](migrate-mealie.md)
