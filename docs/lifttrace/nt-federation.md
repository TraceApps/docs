# NutriTrace federation (LiftTrace side)

Point LiftTrace at a NutriTrace instance and every completed workout automatically posts its calorie burn into NT. That kcal-out then feeds NT's Dynamic and Adaptive calorie-goal modes, so your daily nutrition target reflects the fact that you actually lifted this morning.

For the high-level story and the token model shared across all three apps, see [Federation (cross-app links)](../integrations/federation.md).

## What LiftTrace sends

On workout completion (finishing the last set and marking the session done), LiftTrace posts one summary object per workout:

- **Date** (`YYYY-MM-DD`).
- **Workout name** (free text, e.g. "Push day").
- **Duration** in minutes.
- **Calories burned** (kcal, estimated from the exercises + your body profile).
- **Start time** (ISO 8601, optional).
- **External ID** (`lt:workout:<id>`) so re-posting the same workout updates the NT row in place instead of duplicating. Amending a LiftTrace workout propagates cleanly.

Per-set volume is not part of the wire today. Only the day-level kcal roll-up is what NT's Dynamic / Adaptive TDEE calculation cares about.

The payload lands in NT as `source='lifttrace'` in the `workouts` table, and is rolled up per-day into `wellness_data` under `metric_type='calories_out'` so the cross-source `/api/wellness/calories-out` lookup can find it.

## Configuring it

Mint the token in NutriTrace first: Settings > **API Tokens** > New Token, name it "LiftTrace", tick **write:workouts**, save. Copy the raw `nt_pat_...` value on the confirmation screen; NT only stores the hash after that.

Then in LiftTrace, open Settings and expand **NutriTrace federation**:

1. **Instance URL**: your NT origin, e.g. `https://nutritrace.example.com`. HTTP is allowed for LAN; HTTPS is the sensible default for anything off-LAN.
2. **Access Token**: paste the `nt_pat_...` value.
3. **Test Connection**: LiftTrace calls NT `/api/v1/me` server-side. On success the UI shows "Connected as `<username>`". If the token is missing the `write:workouts` scope the test call fails with a specific message telling you to fix it in NT and re-check.
4. **Enable Federation**: flip on. Save.

The token stays on LiftTrace's server. The Android app / browser never sees it; NT calls are proxied through LiftTrace's `/api/nt/*` routes.

## Wearable priority

If you also have a wearable feeding NT its own kcal-out (Fitbit / Google Health / Garmin / Withings / Health Connect), you get to decide how the two sources combine. This lives on the NT side, under Settings > **Wellness** > **LiftTrace** as **Prefer Wearable Data Over LiftTrace** (`lifttraceOverlapFill`):

- **On (default)**: LT contributes only when no wearable has data for that date. Wearables always win. The natural fill-in semantic: on gym days without a watch, LT covers you; on days you wore the watch, the watch wins.
- **Off**: LT is used as the calories-out source whenever it has data, wearable or not. Pick this if you trust LT's estimate more than your wearable's for lift sessions.

NT never adds the two together. If you want an additive model, keep the wearable off during workouts so LT is the sole source.

## Related

- [Federation (cross-app links)](../integrations/federation.md)
- [Federation API (v1)](../nutritrace/federation-api.md)
- [NutriTrace federation (CookTrace side)](../cooktrace/nt-federation.md)
