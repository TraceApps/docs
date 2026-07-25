# Health Connect (Android)

Health Connect is Android's unified health-data hub. It supersedes the on-device role of Google Fit: any Android app can write to it (Fitbit, Samsung Health, Garmin Connect, Google Fit itself, Gadgetbridge, and dozens of others), and any app you authorize can read from it. NutriTrace's Android app reads Health Connect on the device and forwards the values to the server as wellness records.

The upshot: if your wearable's Android app already pushes to Health Connect, NutriTrace can consume that data **without any developer account, without any OAuth setup, and without any public HTTPS callback.** It is the lowest-friction wellness path Android offers.

## What NutriTrace reads

The Android `HealthConnectSyncWorker` (`android/app/src/main/java/com/nutritrace/app/HealthConnectSyncWorker.kt`) requests granular read permissions for:

- **Steps**
- **Active calories** (feeds Dynamic calorie goal via `/api/wellness/calories-out`)
- **Total calories** (BMR + activity, when your source app writes it)
- **Weight** (feeds Adaptive TDEE)
- **Resting heart rate**
- **Heart-rate samples**
- **Sleep sessions and stages** (feeds Trace Sleep Score)
- **Exercise sessions** (workouts)
- **Blood oxygen (SpO₂)**

Health Connect's permission model is **granular per data type**. You approve each one individually in the Health Connect UI. NutriTrace works with any subset; if you deny sleep but allow steps and weight, only steps and weight sync.

## Setup

### 1. Install Health Connect

- **Android 14 and up**: Health Connect is built into the OS. Nothing to install; open **Settings → Apps → Health Connect** to see it.
- **Android 13 and below**: install [Health Connect](https://play.google.com/store/apps/details?id=com.google.android.apps.healthdata) from the Play Store.

### 2. Wire up your source apps

Open Health Connect and, under **App permissions**, grant your wearable app (Fitbit, Samsung Health, Garmin Connect, Gadgetbridge, and so on) permission to **write** the data types you care about. This step is done inside each source app's settings (usually a "Connect to Health Connect" toggle) rather than in Health Connect itself; Health Connect just shows which apps have been wired up.

### 3. Grant NutriTrace read permissions

**Settings → Wellness → Health Connect** on the NutriTrace Android app has a **Manage permissions** button that jumps into the Health Connect UI scoped to NutriTrace. Approve each data type you want NutriTrace to read. You can change these at any time from Health Connect itself.

### 4. Enable the source in NutriTrace

Toggle **Health Connect** on under **Settings → Wellness**. The sync worker runs on the schedule you configure (`healthConnectSyncMode`, `healthConnectSyncInterval`, and the window fields). Force a sync from the ⟳ button on the Wellness card.

## Data flow

Health Connect reads happen on the phone. The worker packages the readings and pushes them to your NutriTrace server as regular `wellness_data` rows with `source='health_connect'`. On desktop and other devices, those rows are available exactly as if any other wearable had produced them: the Wellness dashboard, `/api/wellness/latest`, and the AI's `get_wellness_data` tool all see them.

In local-only Android mode (no server configured), the same reads land in the on-device SQLite mirror instead. Wellness data is available on the phone, just not synced anywhere.

## Cross-source priority

For calories out, the priority order is `garmin > health_connect > fitbit > lifttrace` (`server/index.js:204-252`). So on days you have both Garmin (via OAuth on the server) and Health Connect readings for the same metric, Garmin's total wins. Weight priority is `withings > fitbit > garmin > health_connect > manual body stats`. Full detail on [Wellness and wearables](wellness.md).

## Gadgetbridge users

If you use [Gadgetbridge](https://gadgetbridge.org/) with a non-official wearable (Amazfit, Xiaomi Mi Band, Huawei Band, and so on), enable its Health Connect integration under Gadgetbridge's settings. Your band's steps, sleep, and heart rate then flow through Health Connect into NutriTrace, no cloud roundtrip required. This is the recommended path for privacy-focused wearable users who don't want a vendor app on the phone at all.

## USER_PREFS vs DEVICE_PREFS

The Health Connect **toggle** and sync-schedule settings are `USER_PREFS` and travel with your account. The **granted permissions** themselves live in the OS, so they are per-device by nature: if you install NutriTrace on a second Android phone, you approve permissions again there.

## Related

- [Wellness and wearables](wellness.md)
- [Fitbit and Google Health migration](google-health.md)
- [Withings](withings.md)
