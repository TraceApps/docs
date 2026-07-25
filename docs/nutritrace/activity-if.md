# Activity and Intermittent Fasting

Two Diary sections that share a page because they answer the same question: what did I do today, outside of eating, that changes the numbers. **Activity** is manual workout logging (with an AI-assisted calorie estimate). **Intermittent Fasting** is a start-stop timer with presets, streaks, and a Statistics card.

## Manual activity logging

Enable **Settings → Diary → Activity section** and an Activity card appears on the Diary. Add a workout with any subset of these fields:

- **Name** (required): "5k run", "Chest day", "Vacuuming".
- **Calories**: kcal burned.
- **Duration**: minutes.
- **Distance**: a free-form string (`5 km`, `3 mi`, `800 m`).
- **MET**: metabolic equivalent, when picked from the 2024 Compendium of Physical Activities. Auto-fills the calorie field via `MET × weight_kg × hours` when you have not typed a calorie value.

Entries land in the `activity_log` table via `POST /api/activity`. Rows marked `is_template=1` become one-tap templates for repeat workouts (a saved "Morning walk" is one tap on the Activity card the next day).

### AI-assisted calorie estimate

Toggle **`activityAutoEstimate`** and a **Trace** button appears next to the calorie field. Describe the workout in plain language ("40 minutes of moderate cycling", "vigorous HIIT session", "easy yoga flow"), tap Trace, and your configured AI provider estimates calories from your body profile (weight, age, gender, activity level) and the description. Review the number before saving; nothing is written until you confirm.

The estimate uses the same AI pipe as Trace Smart Log; requires the AI Assistant to be enabled under **Settings → AI Assistant**.

### Wearable vs manual policy

If a wearable feeds NutriTrace a daily kcal-out total (Garmin, Health Connect, Fitbit via Google Health), that wearable total is authoritative by default. The **`manualActivityPolicy`** setting controls how manual workouts interact:

- **`wearable_wins`** (default): wearable data is authoritative; manual entries are ignored for goal math. Useful when your wearable already captures everything and you log manual entries only for a written record.
- **`manual_wins`**: manual entries take precedence; wearable data is ignored for the day when a manual entry exists.
- **`additive`**: both are summed. Only pick this if you know your wearable does *not* count the manual workout (rare; most modern wearables count basically everything).

The **`calorieAdjustFromActivity`** toggle then decides whether the effective active-calories figure inflates your daily calorie goal (the "eat back exercise calories" pattern) or leaves the goal alone. Off by default.

Endpoint `GET /api/activity/sum/:date?policy=<policy>` returns a breakdown of `{ manual, wearable, effective, policy }` so the Diary header can show "X kcal earned today" without re-doing the math client-side.

## Intermittent Fasting tracker

Toggle **`fastingEnabled`** and the IF widget appears at the top of the Diary. It is a frosted card with a live elapsed timer, a progress bar toward the goal, and a **Start** / **End** button.

### Presets

The built-in presets, and what they mean:

| Preset | Fast : Eat | Common use |
|---|---|---|
| `14:10` | 14h fast, 10h window | Gentle start. |
| `16:8` | 16h fast, 8h window | The default most people land on. |
| `18:6` | 18h fast, 6h window | Tighter window; skip breakfast comfortably. |
| `20:4` | 20h fast, 4h window | Warrior-style. |
| `OMAD` | 23h fast, 1h window | One Meal A Day. |
| `custom` | Any hour count | Set your own via `fastingDefaultHours`. |

You can also define **custom presets** under `fastingCustomPresets` (a list of `{ label, hours }` objects) if you fast on a schedule the built-in presets don't cover.

### Scheduled auto-start

`fastingScheduleEnabled` turns on **scheduled fasts**: pick a time (`fastingScheduleTime`, default `20:00`), pick which days of the week (`fastingScheduleDays`, an array like `[0,1,2,3,4]` for Sun-Thu), and set a goal (`fastingScheduleGoal`, an hour count). A server-side tick fires at your local time on the selected days and starts the fast automatically. `fastingScheduleLastFired` prevents double-firing.

### Notifications

`fastingNotifyOnGoal` posts a device notification the moment you hit the goal. On Android the notification is delivered by native WorkManager (`ReminderWorker.java`); the JS `LocalNotifications` path is intentionally disabled (`_USE_NATIVE_WORKER`) so notifications survive app kills and reboots. See [Notifications and Reminders](../nutritrace/settings.md#notifications) for the full delivery model.

### Statistics card

The IF card on the Statistics page shows:

- **14-day chart**: fast duration per day, with your goal overlaid.
- **Average duration**: rolling average over the range.
- **Longest fast**: your best single fast.
- **Current streak**: consecutive days with a goal-completed fast.
- **Longest streak**: best run.

Backend endpoints (`server/routes/fasts.js`): `GET /api/fasts`, `GET /api/fasts/active`, `POST /api/fasts/start`, `POST /api/fasts/:id/end`, `PATCH/DELETE /api/fasts/:id`. Table `fasts` (`server/db.js:251-266`).

### Trace tool

Ask Trace "what's my current fasting streak?" or "how long was my last fast?" and the `get_fasting_stats` tool returns the current active fast, the last completed fast, current streak, longest streak, and rolling averages. Tool definition lives in `src/lib/aiChat.js`.

## USER_PREFS vs DEVICE_PREFS

Everything on this page is `USER_PREFS`: activity policy toggles, IF enable, presets, schedule, notification preference. All travel with your account across devices, so a fast started on your phone shows on your desktop with the same live timer.

## Related

- [Diary and meal logging](diary.md)
- [Goals and Adaptive TDEE](goals.md)
- [Wellness and wearables](wellness.md)
- [Trace in NutriTrace + Smart Log](trace.md)
