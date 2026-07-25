# Exercise library

LiftTrace comes with a multi-source exercise catalog. Every source is optional, every source can be enabled or disabled independently, and every source is licensed for redistribution. You can also add your own exercises one at a time or bulk-import via spreadsheet.

## The four sources

Sources are registered in `server/exercise-sources/index.js`. Enable and import them from Settings > Integrations > Catalog. Each carries a license badge in the UI so you know what you are handing to your users.

| Source | Count | Media | License | Requires |
|---|---|---|---|---|
| `wger` | ~600 | Sparse images, no GIFs | CC-BY-SA 4.0 | Nothing |
| `free-db` | ~870 | Start/end position images | Public domain | Nothing |
| `exercisedb` | ~1,300 | Animated GIFs | RapidAPI commercial | RapidAPI key |
| `exercisedb-oss` | ~1,500 | Animated GIFs | AGPL-3.0 | Community mirror |

### wger

The wger Workout Manager project's public catalog. Free and open under CC-BY-SA 4.0. Baked into the app; no key required.

### Free Exercise DB

A public-domain GitHub-hosted catalog with a start/end image for every exercise. Good pairing with wger if you want images without touching a commercial API.

### ExerciseDB (RapidAPI)

Commercial catalog reached through RapidAPI. Bring your own key (free tier available), configure it in Settings > Integrations > Catalog > ExerciseDB. The API rotates media URLs weekly, so cached links go stale; re-import every Monday to refresh.

### ExerciseDB (open-source mirror)

A community-hosted AGPL-3.0 mirror of the ExerciseDB dataset, no API key required. The default endpoint is `https://oss.exercisedb.dev`. If you want to self-host the mirror or point at an alternative, set:

```env
EXERCISEDB_OSS_URL=https://your-mirror.example.com
```

The mirror is community-hosted with no SLA. If the endpoint is down, the import fails cleanly and no rows change.

## Seeding on first boot

Set `EXERCISE_SOURCES` at boot to auto-seed one or more sources without touching the wizard:

```env
EXERCISE_SOURCES=wger,free-db
```

Valid IDs: `wger`, `free-db`, `exercisedb`, `exercisedb-oss` (comma-separated, no spaces required). Empty (the default in code) triggers the wizard's exercise-library picker on first launch so a new admin actively chooses. You can re-import a source any time via Settings > Integrations > Catalog > (source) > Import.

## Custom exercises

Create your own from Exercises > **+** > **New Exercise** (or from the Exercise Editor at `src/components/exercises/ExerciseEditor.svelte`). Every custom exercise can carry:

- Name, description, instructions
- Primary and secondary muscle tags
- Equipment tags (including custom ones you have defined)
- An uploaded image, GIF, or video (magic-byte validated on upload)
- A YouTube embed URL

Custom exercises appear inline in the library with a **Custom** chip. They also participate in favorites, search, and the similar-exercise suggestions the same way the seeded ones do.

## Custom XLSX import

For gyms, coaches, or anyone with a spreadsheet-shaped catalog: Exercises > **+** > **Import from XLSX**. The endpoint template is at `GET /api/exercise-import/template`; download it, fill in your exercises, upload the file back.

The importer supports:

- **Named catalogs**: every import row is tagged with a catalog name, so you can enable, disable, or delete a whole catalog at once from Settings > Integrations > Catalog.
- **Equipment tags** as comma-separated lists.
- **Descriptions** and instructions in prose.
- **Media URLs** for images or GIFs (fetched at import time if reachable).

Toggle imported catalogs on and off with `POST /api/exercise-import/catalogs/toggle`, delete with `POST /api/exercise-import/catalogs/delete`.

## Similar exercises

Every exercise detail page suggests substitutes. When you want to swap out an exercise (equipment unavailable, injury, boredom), the "Similar" strip gives you plausible replacements without leaving the page.

The math lives in `src/lib/exerciseSimilarity.js`. Roughly: a candidate's score is `primary_overlap * 3 + secondary_overlap - equipment_difference`, and a candidate needs at least one primary-muscle overlap to be included at all. Custom exercises participate. So do exercises from disabled sources (once imported they are yours).

## Favorites, search, filters

Star any exercise from the detail page or the library grid to promote it to the top of the list. The star persists per user.

Search matches names, aliases (`BB`, `DB`, `BP`, `OHP`, `DL`, `SQ`, `RDL`), and equipment. The equipment filter chip strip is multi-select (v1.0.0-rc.6): pick everything you have access to today and the library filters to only exercises that match. Selection persists across navigation. Custom equipment you added appears with a dashed border once at least one exercise uses it.

## Per-exercise personal records

Each detail page has a Records section: current top weight, current top estimated 1RM, current top volume set, plus a history graph. PRs auto-detect from every completed working set (warm-ups excluded).

## Sharing individual exercises

Every exercise detail page has a **Share** button that produces a portable JSON file. Send it via any channel (email, WhatsApp, Signal, Drive), and the recipient can tap the file to open it directly in LiftTrace, which prompts them to add it. The Exercises page's **+** button also accepts Import from File and Import from URL (raw GitHub links work out of the box; `github.com/blob` URLs auto-rewrite).

Handy for community catalogs: a public repo of JSON files gives everyone a bookmarkable exercise library without a central server.

## Related

- [Diary and set logging](diary.md)
- [Programs and templates](programs.md)
- [Environment reference](../self-hosting/env-vars.md)
