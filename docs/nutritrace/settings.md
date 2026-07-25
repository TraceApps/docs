# NutriTrace settings reference

A map of every section in **Settings**, with a one-line explanation and a pointer to the deeper doc where one exists. Settings render in the order below inside the app (`src/routes/Settings.svelte`).

## USER_PREFS vs DEVICE_PREFS

Two categories of settings, treated differently by the sync layer:

- **`USER_PREFS`** (server-synced) travel with your account across every device you log in from. Nutrition prefs, integrations, notifications, goals, body profile, meal names, unit choices. Defined in `src/stores/settings.js:23-77`.
- **`DEVICE_PREFS`** (local-only) stay on the device that set them. They are never sent to the server and never leave `localStorage` / on-device SQLite. Defined in `src/stores/settings.js:80-88`. Only six of them:
    - `appearance` (light / dark / system, depends on this device's lighting).
    - `navStyle` (bottom-nav / sidebar / both; form-factor specific).
    - `sidebarPersistent` (form-factor specific).
    - `disableAnimations` (performance pref tied to device speed).
    - `barcodeFlashlight` (no flashlight on desktop).
    - `biometricLoginEnabled` (Android-only, per-device biometric unlock).
- **Server-admin-only** keys never reach any client (filtered by `server/lib/server-only-keys.js`): OAuth client secrets, SMTP password, API keys stored server-side.

Rule of thumb when adding a new setting: if it makes sense on every device you own, put it in `USER_PREFS`. If it depends on the form factor or hardware, put it in `DEVICE_PREFS`.

## Profile hero

Top-of-page card with your name, nickname, avatar, DOB, gender, height, weight, target weight, and activity level. Feeds the Mifflin-St Jeor TDEE calculation used by the first-run Wizard and by all Goals math. Deep link: opens `/profile` for editing. All fields `USER_PREFS`.

## Appearance

- **Theme** (`appearance`): light / dark / system. `DEVICE_PREFS`.
- **Accent color** (`accentColor`): preset palette plus custom hex. `USER_PREFS`.
- **Navigation style** (`navStyle`): bottom / sidebar / both. `DEVICE_PREFS`.
- **Start page** (`startPage`): which route the app opens to. `USER_PREFS`.
- **Disable animations** (`disableAnimations`): performance pref. `DEVICE_PREFS`.
- **Page banners** (toggle, style, animation): visual polish. `USER_PREFS`.

## Regional and Units

Language, date format (US / ISO / EU / natural), time format (12h / 24h), timezone. Unit choices for energy (kcal / kJ), weight, height, circumference, distance, temperature. All `USER_PREFS`.

## Diary

Comprehensive Diary UI toggles. Nutrition-bar visibility, totals mode (consumed / remaining), macro legend, brand / timestamp / thumbnail visibility, "show all nutrients", quantity prompt on add, portion-size display, per-day notes, activity-section enable, manual-activity policy (`wearable_wins` / `manual_wins` / `additive`), auto-estimate calories, calorie adjust from activity, quick-calories row, unit metadata, warn on unit mismatch. Also: the **meal-name list** (rename / add / reorder your meal slots) and the IF tracker toggles. All `USER_PREFS`.

See [Diary and meal logging](diary.md) and [Activity and Intermittent Fasting](activity-if.md).

## Water

Daily goal (`waterGoalMl`), unit (`waterUnit`: ml / fl oz), **container library** (add / remove containers with capacity), show-in-Diary and show-in-Statistics toggles. All `USER_PREFS`.

## Foods

Sort orders per tab (`foodsSort`, `mealsSort`, `recipesSort`): favorites-first / alphabetical / recently used / most used. Visibility toggles: categories, labels, notes, thumbnails, yesterday meals + collapsed state, saved-list collapsed state. **Barcode beep** (`barcodeBeep`, `USER_PREFS`). **Crop photos** on upload. **Default food visibility** (`defaultFoodVisibility`) sets whether newly-created foods start private / everyone / specific users. **Biometric login** (`biometricLoginEnabled`, `DEVICE_PREFS`, Android-only).

See [Foods, meals and recipes](foods.md) and [Barcode and Scan Label](scan.md).

## Goals

Calorie goal mode picker (**Fixed** / **Dynamic** / **Adaptive**) plus the Lose / Maintain / Gain factor (0.8 / 1.0 / 1.2). Per-nutrient goals grid. Per-user goal templates. All `USER_PREFS`.

Deep dive: [Goals and Adaptive TDEE](goals.md).

## Body Stats

Reorder / hide the body-stat fields tracked in the Diary (weight, waist, chest, custom). Custom fields defined here render as inputs on the Diary Body Stats card. All `USER_PREFS`.

## Statistics

Chart type, y-axis zero-baseline, average / goal / trend line overlays, include-today, show-empty-days. Reorderable metric chips and hide-metrics list. All `USER_PREFS`.

## Nutrients

Drag-to-reorder every macro and micro, hide the ones you don't want to see, and add custom nutrients (`customNutriments`). The order applies everywhere nutrition renders. All `USER_PREFS`.

## Categories

Food category chips (`foodCategories`). Personal taxonomy for the foods you create. `USER_PREFS`.

## Custom Units

User-defined units on top of the built-in mass / volume set (`customUnits`). Handy for household-specific measures ("scoop", "bar", "cookie"). `USER_PREFS`.

## Connected Services

Per-user integrations for food data:

- **Open Food Facts** (`offEnabled`, `offSearchLanguage`, `offSearchCountry`, `offUploadCountry`, `offImportPortion`, `offUsername`, `offPassword`). Also the admin-only **local mirror** controls (auto-refresh schedule, Refresh Now) which land in server config, not user prefs. See [Open Food Facts](off.md).
- **USDA FoodData Central** (`usdaEnabled`, `usdaApiKey`).
- **Mealie** (`mealieEnabled`, `mealieBaseUrl`, `mealieApiToken`). Token is filtered from client payloads.

See [USDA and Mealie integration](usda-mealie.md).

## AI Assistant

Trace configuration. Enable, provider (Claude / OpenAI / Gemini / OpenAI-compat), API key, model (with `__custom__` slot for models not in the built-in roster), base URL (for `oai-compat`), assistant name (`aiAssistantName`, shared across LiftTrace / CookTrace per the Trace brand rules), goal insights toggle, Smart Log toggle, voice language.

Provider / model / key are `USER_PREFS`; env-locked when the corresponding server-side env var is set (`AI_PROVIDER`, `AI_API_KEY`, `AI_MODEL`, `AI_BASE_URL`), in which case the UI shows a lock badge and the values are read-only.

See [Trace in NutriTrace + Smart Log](trace.md) and the shared [Trace setup page](../trace/setup.md).

## Wellness

Top-level enable (`wellnessEnabled`), sync range, wellness-metrics selection, workouts enable. Sub-sections:

- **Fitbit** / **Google Health**: legacy Fitbit Web API plus successor Google Health API. Client ID / Secret pair per user. Sync mode / interval / window. See [Fitbit → Google Health migration](google-health.md).
- **Withings**: same shape. See [Withings](withings.md).
- **Garmin**: OAuth 1.0a, experimental (partnership-gated).
- **Health Connect** (Android-only, on-device permissions). See [Health Connect](health-connect.md).
- **LiftTrace** (federation source; `lifttraceOverlapFill` toggle).

See [Wellness and wearables](wellness.md).

## Server Connection (Android)

Android-only, appears when the app is in local mode or when you want to change the server it points at. Server URL, credentials, connect, manual sync button, log-out from server, disconnect (return to local mode). Not synced (`DEVICE_PREFS`-adjacent; lives in native prefs).

## Notifications

- **Delivery**: `notifLocalEnabled` (device notifications toggle), `notifPushService` (`apprise` / `gotify` / `ntfy` picker) plus the provider-specific fields (`appriseUrl`, `appriseTag`, `gotifyUrl`, `gotifyToken`, `ntfyUrl`, `ntfyTopic`, `ntfyToken`).
- **Scheduled Reminders**: water (`notifWaterReminders`, `notifWaterInterval`), meal (`notifMealReminders`, `notifMealTimes`), weigh-in (`notifWeighIn`, `notifWeighInTime`), bedtime (`notifBedtime`, `notifBedtimeTime`, `notifBedtimeWindDown`, `notifBedtimeWindDownMin`, `notifBedtimeSmart`).
- **Alerts and Summaries**: goal celebrations (`notifGoalCelebrations`), step goal (`notifStepGoal`), weekly summary (`notifWeeklySummary`, `weeklySummaryDay`, `weeklySummaryTime`), wellness alerts (`notifWellnessAlerts`), workout summary (`notifWorkoutSummary`), sync failures (`notifSyncFailures`).

All `USER_PREFS`. On Android, delivery is via native WorkManager (`ReminderWorker.java`); the JS `LocalNotifications` path is disabled via `_USE_NATIVE_WORKER` so notifications survive app kills and reboots (`BootReceiver.java` re-arms after boot).

## Backup and Restore

**Full Backup** (admin only): create, list, download, restore in-place, upload-and-restore, schedule. Backups are ZIPs of DB + uploads under `BACKUPS_PATH` (defaults inside the uploads volume). OFF local mirror is deliberately excluded.

**Local Full Backup** (Android): offline `.zip` with embedded image files for phone-to-phone transfer with no server involved.

**Portable JSON export / import** (foods / meals / diary / settings, no images).

**CSV diary export**.

**Waistline import** (see [Migrate from Lose It, Cronometer, Waistline](migrate-other.md)).

**Danger Zone**: wipe diary / foods / all data.

See [env vars](env-vars.md) for backup-schedule env vars.

## Import and Export from other apps

Import diary days from MyFitnessPal, Lose It, Cronometer, or generic CSV. Bulk-import custom foods from JSON or CSV. Preview + per-date conflict policy on every commit.

See [Migrate from MyFitnessPal](migrate-mfp.md) and [Migrate from Lose It, Cronometer, Waistline](migrate-other.md).

## Sharing

Current sharing state summary. Bulk share / unshare foods and meals across specific users on the same instance. Row-level grants live in `food_shares` and `meal_shares`. See the Sharing subsection of [Foods, meals and recipes](foods.md).

## Diagnostics

500-line in-app ring buffer log viewer. **Verbose diagnostic logging** toggle. Copy / Share / Clear log. Share log file. Share crash report. **Calibration export** dumps the raw wellness-scoring inputs used to produce the last N scores.

## User Management (admin)

Create the initial admin (setup wizard entry). Invites list, revoke invite. User list, per-user delete / role change / password reset. **Disable user management** (recovery-token gated) is the escape hatch back to single-user mode.

## Authentication (admin)

OIDC provider CRUD (Auth0, Authelia, Authentik, Keycloak, Google, Pocket ID, Custom / Generic templates), test-discovery, per-provider **Auto-Link** and **Auto-Register** toggles, admin group claim / value. Password login enable toggle (env-lockable via `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0` for SSO-only mode).

## Email / SMTP (admin)

Host, port, TLS toggle, user, pass, from address. Test-email button. Server-side settings; UI fields are env-locked when the corresponding `SMTP_*` env vars are set.

## API Tokens (admin)

Federation API tokens: create with name and scopes (`read:foods`, `write:workouts`, `read:me`, ...), list, delete. Hashed with SHA-256 (raw token shown once at creation). See [Federation API](federation-api.md).

## About

Version, links, license (AGPL-3.0), credits.

## Related

- [Environment reference](env-vars.md)
- [First-run Wizard walkthrough](../getting-started/first-run.md)
- [OIDC provider setup](../auth/oidc.md)
