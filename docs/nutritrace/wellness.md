# Wellness & wearables

The Wellness page is where wearable data lives inside NutriTrace: last night's sleep, today's readiness, this week's resilience bucket, HRV, resting HR, steps, SpO₂, workouts. Four provider paths feed it (Fitbit via Google Health, Withings, Garmin, and Android Health Connect), and the same data feeds Dynamic calorie goals, Adaptive TDEE weight priorities, and the Trace AI's `get_wellness_data` tool.

## What renders on the page

For each connected provider, a **live connection status banner** sits at the top of its card:

- **Green**: authorized and last sync succeeded.
- **Yellow**: authorized but the last sync errored. The actual error text renders underneath so you can act on it.
- **Red**: not connected.

Re-authorize, force-sync, and disconnect all live in the same banner, so the connection state and the controls that affect it never drift out of sync.

Below the banners: metric tiles with sparklines. Sleep score, sleep-quality sub-metrics (**Sound Sleep**, **Restlessness**, **Time to Sound Sleep**, **Interruptions**), Trace **Readiness**, Trace **Resilience** (**Optimal / Balanced / Low**), HRV, RHR, steps, SpO₂, and today's workouts. Which tiles show is configurable per user under **Settings → Wellness → Wellness metrics selection**.

## Trace scores (computed by NutriTrace)

Some providers publish their own sleep or readiness scores; NutriTrace prefers those. Where they don't, NutriTrace computes its own. The computed scores carry the **Trace** prefix to keep the distinction obvious.

- **Trace Sleep Score** combines sleep duration, deep and REM percentages, SpO₂, HRV, and efficiency into a single 0–100 value.
- **Trace Readiness** weighs HRV against a 30-day baseline plus resting HR and last night's sleep, with an activity-spike penalty.
- **Trace Resilience** replaces the older numeric stress score with a categorical bucket computed from three pillars: **Physical Calmness** (HRV + RHR vs baseline), **Activity Balance** (acute-to-chronic workload ratio over 7 vs 28 days), and **Sleep Patterns**.

Formulas live in `server/lib/wellness-scores.js` and `server/lib/google-health.js`. The Diagnostics screen has a **Calibration export** that dumps the raw inputs used to produce the last N scores, in case you want to eyeball the math against your own numbers.

## Cross-source `/latest`

Wearables often overlap. If your weight comes from Withings and your steps come from Fitbit and your sleep comes from Garmin, you don't want three separate queries per widget. `GET /api/wellness/latest?metric=<name>` picks the freshest available value across all connected providers for that metric.

Under the hood this walks each connected source in a stable priority order, takes the freshest timestamp within the query window, and returns the value plus the winning source. The Diary macro bar's "weight today" reading and the Adaptive TDEE weight input both use this.

## Cross-source `/calories-out`

`GET /api/wellness/calories-out?date=<diary date>` is the source of the daily burn used by **Dynamic** calorie goals. Priority order is fixed at:

```
garmin > health_connect > fitbit > lifttrace
```

The `lifttraceOverlapFill` toggle (**Settings → Wellness → LiftTrace**) decides whether LiftTrace-reported workout kcal fills in on days where a wearable-reported total is missing. On days with both, the wearable total wins by default; enabling overlap-fill treats the LiftTrace kcal as additive when the wearable session obviously didn't cover it. See the [federation API](federation-api.md) for how LiftTrace posts here.

## Wellness source gates (implementation note)

Any UI predicate that asks "does the user have Fitbit connected?" or "is any wearable configured?" **must** use the shared derived stores exported from `src/stores/settings.js` (`fitbitFamilyEnabled`, `anyWearableConfigured`, and friends). Inline disjunctions like `fitbitEnabled || googleHealthEnabled || garminEnabled || healthConnectEnabled` are a recurring source of state-drift bugs (issues NT #57 / #62 / #65) because a new provider slot needs updating in every disjunction site, and one of them always gets missed.

If you're contributing UI code that gates on wearable presence, look up the existing derived store first. If your case isn't covered, add a store rather than open-coding a check.

## Android: no API setup prompts

On the Android app, the Fitbit / Withings / Garmin **Client ID and Secret** fields do not surface. Those integrations require a public HTTPS callback that a phone-only, LAN-only installation cannot satisfy, so the app hides the developer-setup UI entirely and uses generic device-oriented messaging instead ("connect on the web to sync from Fitbit"). **Health Connect** is the wearable path Android surfaces natively; it is on-device, needs zero server setup, and reads steps, sleep, heart rate, weight, and exercise directly from any HC-participating app on the phone.

If you want Fitbit on Android too, the Android app in server-connected mode pulls whatever the server has already synced; you just complete the Google Cloud OAuth setup on the web side, and the phone sees the data via the normal sync.

## Provider setup pointers

Each provider has its own page for the developer-account setup and the OAuth details:

- [Fitbit → Google Health migration](google-health.md): the walkthrough most existing users need, with the Google Cloud OAuth setup.
- [Withings](withings.md): free developer account, OAuth 2.
- [Health Connect (Android)](health-connect.md): on-device permissions.

Garmin uses OAuth 1.0a and requires a Health API partnership from Garmin (not a free developer program). If you have access, the callback is `/api/wellness/garmin/callback`. Marked Experimental in the UI.

## USER_PREFS vs DEVICE_PREFS

Wellness config is **user-synced** (provider enablement, cross-source priorities, metric selection). Health Connect on Android is by nature per-device (the OS permissions live on the phone), but the toggle for whether to use HC as a wellness source syncs with the rest of your account.

## Related

- [Fitbit → Google Health migration](google-health.md)
- [Withings](withings.md)
- [Health Connect (Android)](health-connect.md)
- [Goals & Adaptive TDEE](goals.md)
- [Federation API (v1)](federation-api.md)
