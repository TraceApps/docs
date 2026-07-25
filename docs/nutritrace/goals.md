# Goals & Adaptive TDEE

Goals set the calorie target and per-nutrient targets that everything else on the Diary compares against: the macro bar, the remaining-calories count, the Statistics goal overlay, the reminders. NutriTrace supports three calorie modes and a full per-nutrient goal grid, with template support so different phases (cut / maintain / bulk) are one tap apart.

## The three calorie modes

Under **Settings → Goals → Calorie Goal Mode** (`calorieGoalMode`) you pick one of three:

### Fixed

You type a target. Simple, unchanging. Good when you already know your maintenance and are running a straightforward cut or bulk, or when your wearables situation is not stable enough to trust anything else.

### Dynamic

Yesterday's wearable-measured calories out, multiplied by `calorieGoalFactor` (three-segment picker: Lose = 0.8, Maintain = 1.0, Gain = 1.2). Requires a wearable that reports a `calories_out` metric.

The source for the daily burn is `GET /api/wellness/calories-out?date=<diary date>`, which walks a priority order: **garmin > health_connect > fitbit > lifttrace**. The order is flippable at the LiftTrace slot via `lifttraceOverlapFill`; the rest is fixed. If yesterday's data has not synced yet, the mode falls back to your Fixed goal for the day and re-computes once the sync lands.

Dynamic works well when your daily burn genuinely varies day-to-day (real training weeks, active job) and poorly when your wearable under-reports or the sync is flaky. Watch the Wellness page's per-provider banner to make sure the source you expect is the one being used.

### Adaptive

A learned TDEE from your last 35 days of intake and weight, MacroFactor-style. This is the mode most people want once they have three or four weeks of data.

**The math.** The algorithm lives in `server/lib/adaptive-tdee.js`. It pulls 35 days of:

- **Daily intake**: sum of food calories from the diary.
- **Daily weight**: priority: **Withings > Fitbit > Garmin > Health Connect > manual body stats**. Sparse weights are linearly interpolated between known measurements.

It runs a linear regression on the weight series to find the trend (kg per day), converts that trend into a daily energy balance using `1 kg ≈ 7,700 kcal` (the standard textbook value; conservative for pure-adipose change, close enough for mixed lean+fat), and computes:

```
TDEE = average daily intake − daily energy balance
```

That learned TDEE is then multiplied by your goal factor (Lose −20%, Maintain, or Gain +20%) to produce the daily target that shows on the Diary bar and the Statistics goal overlay. Exposed constants in the source: `KCAL_PER_KG = 7700`, `DEFAULT_WINDOW_DAYS = 35`, `MIN_VALID_DAYS = 21`.

**The readiness gate.** Adaptive stays in fallback (using your Fixed goal) until you have **at least 21 days** where both weight and diary are logged after interpolation. 21 is enough signal to detect a real trend through day-to-day noise without forcing 35 perfect days. The Goals page renders a readiness card with a progress bar toward that 21-day minimum.

**The confidence percentage.** Once ready, the same card shows the learned TDEE and a confidence percentage based on data coverage and how cleanly your weight trends. A low confidence value means treat the number as an approximation, not a prescription. The endpoint is `GET /api/goals/adaptive-tdee` if you want to hit it from a script.

**For best results.**

- **Weigh frequently.** Daily is best; every other day is fine. A connected scale automates this. Sparse weights get interpolated, but more real measurements produce a tighter estimate.
- **Log food consistently.** Days with no diary entries are dropped from the calculation. Skipping half your days makes the estimate noisy.
- **Don't switch goal factors mid-window.** The rolling 35-day average converges over time. Weekly bouncing between cut and bulk lags the learned TDEE.
- **Re-weigh in similar conditions.** First thing in the morning, after using the bathroom, before food or drink. Daily fluctuations are mostly water; the trend is what matters.

**Caveats.**

- Big changes in activity (started running, broke a leg) take roughly two weeks to fully reflect in the rolling window.
- Travel weeks with under-logged food can pull the estimate up.
- Body composition matters: 7,700 kcal/kg is accurate for adipose tissue; heavy recomp (fat down, muscle up in similar amounts) can deviate. The estimate is calibrated to body-mass change, not fat-mass change specifically.

## Per-nutrient goals

Below the calorie mode picker, the same Goals page lists a target for every nutrient in your configured order (protein, carbs, fat, plus any micros or custom nutrients you have visible). Numeric targets, absolute or percent-of-calories for the macros, plain numeric for micros. The Diary and Statistics pages both surface these as overlays.

## Goal templates

Templates are named snapshots of the full goal grid, stored under `goalTemplates` in your user settings. Save the current state as "Cut", "Maintain", or "Bulk", then swap between them without re-typing every field. Handy when you run distinct phases and want to move between them cleanly.

## The Wizard

**Settings → About → Run Wizard** (or `/wizard` directly) reopens the first-run flow. It collects gender, date of birth, height, weight, target weight, and activity level, then computes a starting TDEE via the **Mifflin-St Jeor** formula:

```
Male:   BMR = 10·weight_kg + 6.25·height_cm − 5·age + 5
Female: BMR = 10·weight_kg + 6.25·height_cm − 5·age − 161
TDEE  = BMR × activity_factor
```

Activity factor comes from the picker (sedentary / lightly active / moderately active / very active / extra active). The Wizard also proposes a water goal (roughly 35 ml per kg body weight, rounded) and offers to enable multi-user or leave the instance single-user.

## USER_PREFS vs DEVICE_PREFS

Goals are **user-synced** (`USER_PREFS`, `src/stores/settings.js:23-77`). Your calorie mode, factor, per-nutrient targets, and template list follow your account across devices. Nothing goals-related is per-device, so a change on the web reflects on the phone within one sync cycle.

## Related

- [Diary & meal logging](diary.md)
- [Wellness & wearables](wellness.md)
- [Activity & Intermittent Fasting](activity-if.md)
- [Fitbit → Google Health migration](google-health.md)
