# Health Connect (Android)

Health Connect is Android's unified health-data hub. It supersedes the on-device role of Google Fit: any Android app can write to it (Fitbit, Samsung Health, Garmin Connect, Google Fit itself, Gadgetbridge, and dozens of others), and any app you authorize can read from it. NutriTrace's Android app reads Health Connect on the device and forwards the values to the server as wellness records.

The upshot: if your wearable's Android app already pushes to Health Connect, NutriTrace can consume that data **without any developer account, without any OAuth setup, and without any public HTTPS callback.** It is the lowest-friction wellness path Android offers.

## What NutriTrace reads

NutriTrace asks Health Connect for read access to:

- **Steps**, **distance** and **floors climbed**
- **Active calories** and **total calories** (BMR + activity, when your source app writes it; feeds the Dynamic calorie goal via `/api/wellness/calories-out`)
- **Exercise sessions** (workouts, and the day's active minutes)
- **Hydration** (water)
- **Heart rate** (the day's average) and **resting heart rate**
- **Heart rate variability (HRV)**, for Trace Readiness and Resilience (see [HRV](#hrv))
- **Sleep sessions and stages**
- **Blood oxygen (SpO₂)**, **respiratory rate**, **VO₂ max**, **blood pressure** and **body temperature**
- **Weight** (feeds Adaptive TDEE), **body fat**, **bone mass**, **lean body mass** and **basal metabolic rate**

Health Connect's permission model is **granular per data type**. You approve each one individually in the Health Connect UI. NutriTrace works with any subset; if you deny sleep but allow steps and weight, only steps and weight sync.

Health Connect only holds what your apps write into it. It has no place for vendor scores, so Garmin's sleep score, Body Battery and stress, or Fitbit's Active Zone Minutes, don't come through it. [Wellness and wearables](wellness.md#what-each-source-gives) compares what each source gives.

### HRV

Health Connect keeps HRV (RMSSD) as separate readings with a time, while Fitbit and Garmin send one value per night. A day's HRV in NutriTrace is the average of the readings taken during the night you woke up from that day, the same night its sleep goes to. When that night has no readings (no sleep recorded, the watch was off at night, or sleep isn't allowed), it is the average of that day's readings, leaving out any taken during another day's night.

Trace Readiness needs HRV on 3 nights before it shows a score. Only apps that write HRV into Health Connect can fill it; check your watch app's Health Connect settings if HRV stays empty.

## Setup

### 1. Install Health Connect

- **Android 14 and up**: Health Connect is built into the OS. Nothing to install; open **Settings → Apps → Health Connect** to see it.
- **Android 13 and below**: install [Health Connect](https://play.google.com/store/apps/details?id=com.google.android.apps.healthdata) from the Play Store.

### 2. Wire up your source apps

Open Health Connect and, under **App permissions**, grant your wearable app (Fitbit, Samsung Health, Garmin Connect, Gadgetbridge, and so on) permission to **write** the data types you care about. This step is done inside each source app's settings (usually a "Connect to Health Connect" toggle) rather than in Health Connect itself; Health Connect just shows which apps have been wired up.

### 3. Enable the source and allow the data types

Turn on **Enable Health Connect** under **Settings → Wellness → Health Connect** on the NutriTrace Android app. Health Connect then asks, on one screen, which data types NutriTrace may read; allow the ones you want. You can change these at any time from Health Connect itself, under **App permissions → NutriTrace**.

If you turned Health Connect on before NutriTrace read a data type (HRV came later), or you left some off, the same card shows **Data Not Allowed Yet** with how many there are and an **Allow** button that asks again for just those. A type you allow after the first sync gets the last 7 days with the next catch-up (see [Past days and catch-up](#past-days-and-catch-up)), enough for Readiness, which needs 3 nights. Sync keeps working with whatever is allowed, so you can leave any of them off.

### 4. Choose when it syncs

The sync worker runs on the schedule you configure (`healthConnectSyncMode`, `healthConnectSyncInterval`, and the window fields). Force a sync from the ⟳ button on the Wellness card; on a past day, it reads that day (see [Past days and catch-up](#past-days-and-catch-up)).

### 5. Allow reading in the background

When you turn Health Connect on, NutriTrace also asks to **access data in the background**. With it, the hourly background sync reads Health Connect while NutriTrace is closed, so the day's values reach your server without opening the app. Without it, Health Connect only answers NutriTrace while the app is open: everything still syncs, just when you open the app. Workouts, floors and water come through the same reader, which also runs once each time you open NutriTrace or bring it back to the front (not again within five minutes) and when you tap Health Connect Sync; Wellness refreshes when it's done. **Settings → Wellness → Background Sync** says when background reading isn't allowed and has an **Allow** button; you can also change it in Health Connect under **App permissions → NutriTrace**. On a phone whose Health Connect can't read in the background, Settings says so and the app reads while it's open.

## Data flow

Health Connect reads happen on the phone. The worker packages the readings and pushes them to your NutriTrace server as regular `wellness_data` rows with `source='health_connect'`. On desktop and other devices, those rows are available exactly as if any other wearable had produced them: the Wellness dashboard, `/api/wellness/latest`, and the AI's `get_wellness_data` tool all see them.

In local-only Android mode (no server configured), the same reads go into the on-device SQLite mirror instead. Wellness data is available on the phone, just not synced anywhere.

## Past days and catch-up

Data often reaches Health Connect late: a watch that syncs the next morning, a night of sleep you edit a day later, a phone that was off for a few days. NutriTrace reads past days too, and each day's values go to that day: steps and calories for the day, the latest weight and heart-rate readings taken that day, and a night of sleep, and its HRV, on the day you woke up.

- **The first time** NutriTrace reads Health Connect (or the first time after updating), it reads the **last 30 days**. That's all Health Connect lets an app read from before you gave it access, so older history can't come in this way.
- **After that**, the app catches up on the **last 7 days** at most once an hour: when it syncs with your server (as it opens, when it comes back to the front, or when you tap Sync), when you tap Sync on the Wellness page or in **Settings → Wellness**, and in the hourly background sync when reading in the background is allowed (see [step 5](#5-allow-reading-in-the-background)). Without a server, open the Wellness page or tap Sync there. If the app was away for longer than a week, it reads back to the last day it covered, up to 30 days.
- **The sync every 30 seconds** while the app is open and connected to your server reads today only.
- **To pull in a past day right away**, open that day on the Wellness page and tap the Health Connect sync button.

Only values that changed are sent to your server, so reading a day again costs nothing when nothing in it changed. A value deleted in Health Connect is not removed from NutriTrace.

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
