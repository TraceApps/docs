# Workout history import

If you have been logging elsewhere, LiftTrace can absorb the history. Five sources are supported: Strong, Hevy, FitNotes, Jefit (all CSV), and Garmin FIT (raw `.fit` file). Every import runs a two-step preview then commit, fuzzy-matches exercise names against your library, and lets you choose skip-versus-replace on duplicate dates.

## The general flow

1. Export your history from the source app (steps per source below).
2. In LiftTrace, open **Settings > Data > Workout Import**.
3. Pick the source.
4. Upload the export file.
5. Review the preview: number of workouts, number of matched vs unmatched exercises, dates that conflict with existing LiftTrace workouts.
6. Choose **skip** or **replace** on duplicate-date collisions.
7. Commit.

Unmatched exercise names persist as free-text and stay attached to their sets. You can relink them to library entries later from the exercise picker.

## What does not survive

- **RPE** is not in Strong, Hevy, FitNotes, or Jefit exports. Every imported set arrives without RPE regardless of source. You can back-fill from memory on individual sets after import.
- **Superset grouping** is not exported by Strong, FitNotes, or Jefit. Sets from a superset arrive as ungrouped consecutive exercises. Hevy exports superset markers; the Hevy adapter reconstructs the grouping.
- **Rest-timer values** are exported by some sources but LiftTrace does not currently store per-set rest values on imported workouts.
- **Notes** carry through when the source exports them.
- **Body weight** and body-stat entries are not part of workout export files; log those separately.

## Per-source instructions

??? details "Strong (iOS / Android)"
    **Export from Strong**

    1. Open Strong.
    2. Go to Profile > Settings > Export Data (or Settings > Backup).
    3. Choose CSV. Send the file to yourself (email, AirDrop, Files app).

    **Import to LiftTrace**

    1. Settings > Data > Workout Import.
    2. Pick **Strong** from the source picker.
    3. Upload the `.csv` file.
    4. Review the preview and commit.

    Notes: Strong does not export supersets or RPE. Rest values are exported but not persisted.

??? details "Hevy (iOS / Android)"
    **Export from Hevy**

    1. Open Hevy.
    2. Go to Profile icon > Settings > Export Data.
    3. Choose CSV. Save or send the file.

    **Import to LiftTrace**

    1. Settings > Data > Workout Import.
    2. Pick **Hevy**.
    3. Upload the `.csv`.
    4. Review and commit.

    Notes: Hevy exports superset markers, so the Hevy adapter reconstructs superset groupings during import. RPE is not exported.

??? details "FitNotes (Android)"
    **Export from FitNotes**

    1. Open FitNotes.
    2. Menu (top-left) > Export.
    3. Choose CSV (not the SQLite backup; the CSV is what the importer reads).
    4. Save the file to a location you can move to your LiftTrace instance.

    **Import to LiftTrace**

    1. Settings > Data > Workout Import.
    2. Pick **FitNotes**.
    3. Upload the `.csv`.
    4. Review and commit.

    Notes: FitNotes does not track supersets or RPE at all, so nothing is lost that was ever there.

??? details "Jefit (iOS / Android)"
    **Export from Jefit**

    1. Open Jefit.
    2. Profile > Settings > Data Management > Export Log.
    3. Choose CSV.

    **Import to LiftTrace**

    1. Settings > Data > Workout Import.
    2. Pick **Jefit**.
    3. Upload the `.csv`.
    4. Review and commit.

    Notes: Jefit does not export supersets or RPE.

??? details "Garmin FIT (Garmin Connect)"
    **Export from Garmin Connect**

    1. Open Garmin Connect on desktop web (mobile does not offer per-activity export).
    2. Go to Activities.
    3. Filter to a strength-training activity.
    4. Open the activity, click the gear icon (top-right), choose **Export Original**. The download is a raw `.fit` file.
    5. Repeat per activity, or use `garmin-connect-export` scripts to pull multiple.

    **Import to LiftTrace**

    1. Settings > Data > Workout Import.
    2. Pick **Garmin FIT**.
    3. Upload the `.fit` file.
    4. Review and commit.

    Notes: LiftTrace parses sets, reps, weights, and rest periods from the FIT strength-training schema. Non-strength activities (runs, rides, swims) are skipped cleanly. RPE and supersets are not part of the Garmin FIT strength schema. Import one activity per file today; batch import for a folder of `.fit` files is on the roadmap.

## Fuzzy exercise matching

Every adapter normalizes to a canonical shape (`{ date, name, notes, duration_min, exercises: [{ sourceName, sets, superset_id, superset_size }] }`), then the importer matches `sourceName` against your library using the same fuzzy pipeline the Smart Log parser uses:

1. Exact name match.
2. Starts-with match.
3. Substring match.
4. Token-overlap match.
5. Fallback: keep the source name as free text.

If a source-app exercise does not match anything in your library, it does not block the import; the set lands with the raw name and shows up in the diary the same way any exercise does. You can relink it to a library entry from the exercise's row menu > **Rename / Relink**.

## Duplicate-date behavior

If the import preview sees a workout on a date you already have logged in LiftTrace, you get a per-workout choice:

- **Skip**: keep your existing LiftTrace workout, drop the imported one.
- **Replace**: overwrite the existing LiftTrace workout with the imported one.

The default is skip so you cannot accidentally clobber sets you logged natively. Choose per row; there is no bulk-replace switch, on purpose.

## What is not supported

- **Apple Health**: no strength-training schema, so there is nothing structured to import. Manual log or Smart Add is the workaround.
- **Google Fit** strength: deprecated by Google. Not supported.
- **Other proprietary formats**: file a request with a sample export and the adapter table can grow.

## Related

- [Diary and set logging](diary.md)
- [Exercise library](exercises.md)
