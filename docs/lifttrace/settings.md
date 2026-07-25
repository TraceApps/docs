# LiftTrace settings reference

Every LiftTrace setting, grouped the way the Settings page groups them, with a one-line summary and a pointer to the page that goes deep. Per-user settings are visible to every account. Admin-only sections only show up for the `admin` role and only in server mode (they are hidden entirely on standalone Android in local mode).

Search inside Settings is powered by a keyword index at `src/routes/Settings.svelte:360`; if a setting is missing from search after an update, that index needs a same-commit entry.

## Profile

Top-of-page hero on Settings. Links out to `/profile`.

- **Full name / Nickname** display name across the app.
- **Email** used for password reset and SMTP notifications.
- **Birthday, gender, height, current weight** feed BMR estimates and the calorie-estimate overlay.
- **Avatar** upload; validated with magic-byte checks.
- **Change password / Sign out**.

## Display

### Appearance

- **Theme** Light / Dark / System.
- **Accent color** twelve presets (Iron, Mint, Blue, Red, Purple, Teal, Gold, Indigo, Pink, Rose, Cyan, Lime) plus a custom hex picker.
- **Navigation style** bottom bar, sidebar, or both.
- **Persistent sidebar** keep the sidebar pinned on desktop.
- **Start page** Diary, Exercises, Programs, Statistics, Radio, or Coaching.
- **Reduce motion** cuts animations and Trace face movement.
- **Goal celebrations** enable the streak / PR / goal-reached confetti moments.
- **Page banners** the accent-tinted headers on top of each page, with an animation-style sub-toggle (static / gradient / animated).

### Units

- **Language** locale picker (see the i18n status in the About page).
- **Weight unit** lbs or kg. Body-stats, prescriptions, and charts follow.
- **Date format** US `MM/DD/YYYY`, EU `DD/MM/YYYY`, or ISO `YYYY-MM-DD`.
- **Time format** 12-hour or 24-hour.

## Data and tracking

### Workout

- **Weekly goal** target workouts per week; feeds streaks and Overview.
- **Screen keep-awake** wake-lock during workouts (part of [Diary and set logging](diary.md)).
- **Show calorie estimate** overlays estimated kcal per session.
- **Auto-fill last weights** prefill new sets from the last completed working set.
- **Completion summary** show the 1080x1350 share card on workout finish.
- **Auto-collapse completed exercises**, **Auto-name workouts**, **Reorder method** drag handle or arrow buttons, **Confirm before removing**.
- **Auto-generate warm-up sets** insert a ramp for each working set. Generator in `src/lib/workout.js`.
- **Track RPE per set** show the RPE column on every set row.
- **Rest timer** on / off, plus **Rest duration**, **Auto-start on set**, **Vibrate**, and **Tone** as separate toggles.

### Statistics

- **Default chart type** bar or line for muscle-group breakdowns.
- **Lock Y-axis to zero** honest comparisons across date ranges.
- **Show average line**, **Show trend line** overlays on progression charts. See [Statistics and PRs](statistics.md).

## Integrations

### Catalog

Server mode only. Per-source enable / disable / import / clear for `wger`, `free-db`, `exercisedb` (RapidAPI key field), and `exercisedb-oss`. Per-imported-catalog enable / disable / delete. Offline exercise library toggle (Android only, controls image cache). Full detail in [Exercise library](exercises.md).

### Trace

- **Enable AI assistant** master switch for Trace.
- **Provider** Claude, OpenAI, Gemini, or OpenAI-Compatible.
- **Base URL** required for `oai-compat`, ignored otherwise.
- **Model** dropdown of presets plus **Custom** for any model ID the vendor supports.
- **Custom model ID** free-text.
- **API key** masked; a **Change** button appears once stored.
- **Assistant name** rename Trace to whatever fits your setup.

Env-locked and read-only when any `AI_*` env var is set. See [Trace setup](../trace/setup.md).

### Radio

- **Self-hosted music** on / off.
- **Streaming stations** on / off.
- **Provider** Subsonic, Jellyfin, Plex, or Emby.
- **Server URL, Username / Plex token, Password / Emby API key**.
- **Crossfade** 1 to 12 seconds.
- **Highest quality playback** bit-perfect toggle, falls back to 320 kbps MP3 on mixed codecs.

Full walkthrough in [Radio player](radio.md).

### Federation

- **Enable federation**, **Instance URL** (target NutriTrace instance), **Access token**, **Wearable priority**. Posts completed workouts to NutriTrace so calorie balance updates automatically. See [NutriTrace federation](nt-federation.md).

## App

### Server connection (Android only)

Inline block near the top of Settings on the native app. Shows Connected, URL, Last synced, **Sync now**, **Log out**, **Disconnect**. When disconnected, offers URL / Username / Password / **Connect**. First-time connect with local data present shows an **Upload / Download / Merge** dialog.

### Notifications

- **Device notifications** browser or OS notification permission.
- **Push service** None, Gotify, ntfy, or Apprise; each brings provider-specific URL / token / topic / tag fields.
- Per-alert toggles: **Workout reminder** (with time), **Rest-day reminder**, **Streak alert** (with time), **Workout complete**, **Personal records**, **Coach feedback**.
- Trainer-only alerts: **Member completes prescribed**, **Member missed a prescription**, **Member replied to feedback**. See [Coaching](coaching.md#notifications-for-coaches).
- **Weekly summary** with delivery day and time.
- Android adds battery-optimization, low-battery, sound, and channel controls.

### Backup and restore

- **Full backup** create, download, restore, delete, upload-restore (server mode only, admin).
- **Auto-backup schedule** off / daily / weekly / monthly plus time and retention count (all modes).
- **Clear all settings** wipes preferences (destructive).
- **Clear all data** wipes workouts and body stats (destructive).

### Workout import

Picker for Strong, Hevy, FitNotes, Jefit, or Garmin FIT; upload file, preview, skip / replace on duplicate dates, commit. See [Workout history import](import.md).

### Help and improve / Diagnostics

- **Verbose diagnostic logging** toggle.
- **View diagnostic logs** in-memory log viewer with a copy-to-clipboard button for bug reports.

## Admin

Only visible to `admin` in server mode.

### User management

- **Users list** with role picker (`admin` / `trainer` / `member`), **Assign trainer**, **Reset user password**, **Delete user**.
- **Invite user** email (SMTP) or shareable link.
- **Require strong passwords** enforces zxcvbn score >= 3.
- **Session duration** hours; capped by `MAX_SESSION_HOURS`.
- **Disable user management** danger; drops the app back to zero-user mode.
- **Enable user management** activates multi-user mode from a single-user install; creates the first admin.

### Authentication

- **OIDC providers** list (add / edit / delete / test). Per provider: Display name, Issuer URL, Client ID, Client secret (encrypted at rest with AES-GCM + HKDF), Redirect URIs, Scope, Token endpoint auth method, Admin group claim + value, Auto-link, Auto-register, Active.
- **Allow password login** server-wide toggle; env-lockable via `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN`.
- Env-defined providers show up read-only with a lock badge. Full recipes: [OIDC single sign-on](../auth/oidc.md).

### Email / SMTP

- **SMTP host, port, secure, username, password, from address**.
- **Send test email** with recipient picker and a real branded HTML delivery.

Env-locked when `SMTP_HOST` is set. See [SMTP setup](../integrations/smtp.md).

## About

Version, license (AGPL-3.0), sister-app link to NutriTrace, and Ko-fi link. `SettingsAbout.svelte` is also the entry point for checking installed version against the currently released tag.

## Related

- [Environment reference (LiftTrace)](env-vars.md)
- [Shared env vars](../self-hosting/env-vars.md)
- [Trace setup](../trace/setup.md)
- [OIDC single sign-on](../auth/oidc.md)
- [SMTP setup](../integrations/smtp.md)
