# Garmin

NutriTrace can read your Garmin data two ways:

- **Personal API Key** (recommended). A free key you create yourself in Garmin Connect. No developer account, no approval, and no public HTTPS address needed, because NutriTrace's server asks Garmin for the data (Garmin never calls back). Each person on a NutriTrace server adds their own key.
- **Health API Partner**. Garmin's business program (OAuth 1.0a, push). It needs an approved partnership, which most self-hosters can't get. If you already have one, it keeps working as before; see [Health API partner setup](#health-api-partner-setup).

Both write the same readings, so Wellness, Statistics, Goals, Dynamic calorie goals, Adaptive TDEE, Readiness, Resilience, alerts, Trace and the weekly email treat them the same.

## Get a Personal API Key

1. On a computer, sign in to [Garmin Connect](https://connect.garmin.com) and open **Settings > Personal API Key** (or go straight to [connect.garmin.com/app/settings/personalApiToken](https://connect.garmin.com/app/settings/personalApiToken)).
2. Select **Create Key**, accept Garmin's terms, and choose how long the key lasts: 30, 90 or 180 days, or one year.
3. Copy the key. Garmin shows it only once; Garmin Connect then lists it as **Active**, with the date it expires.

Garmin's terms describe the Personal API as access for one person to their own data. Use your own key for your own account.

## Connect it in NutriTrace

**Settings > Wellness > Garmin**:

1. Turn on **Enable Garmin**.
2. Under **How to Connect**, choose **Personal API Key**.
3. Paste the key. Optionally set **Key Expires On** to the date Garmin shows next to the key (the **30 Days**, **90 Days**, **180 Days** and **1 Year** buttons count from today). Leave **Account Region** on **Global** unless your Garmin account is a China account (garmin.cn).
4. Select **Connect**. The server checks the key with Garmin right away. If Garmin doesn't accept it, the card says so and nothing is saved: copy the whole key again, or check it hasn't expired or been deleted in Garmin Connect.

Once connected, the card shows **Linked**, when it last synced, **Sync**, **Disconnect**, the key's expiry date, **Change Date** and **Replace Key**. The first sync starts right away and gets the last 30 days.

!!! note "No watch on the account yet?"
    That's fine. The card says it's connected with no Garmin data yet, and readings show up after your watch syncs with Garmin Connect.

## What it reads

| Garmin data | NutriTrace metric |
|---|---|
| Steps, step distance, floors climbed | Steps, Distance, Floors Climbed |
| Total and active calories burned | Calories Burned, active calories |
| Moderate and vigorous intensity minutes | Moderate Intensity, Vigorous Intensity, and their sum as Active Minutes |
| Resting and max heart rate | Resting Heart Rate, Max Heart Rate |
| Average stress, Body Battery high and low | Avg Stress, Body Battery (Peak), Body Battery (Low) |
| Average SpO₂ (the night's average when the day has none) | SpO2 |
| Sleep time, deep, light, REM and awake time, sleep score, bed and wake times, average breathing rate | Sleep Duration, stages, Sleep Score, sleep timing, Respiratory Rate, all filed under the morning you woke up |
| Last night's average HRV | HRV (RMSSD) |
| Weight and body composition from a Garmin scale (or weight entered in Garmin Connect) | Weight, BMI, Body Fat, Body Water, Muscle Mass, Bone Mass, Visceral Fat, Metabolic Age |

The **Body** tab on the Wellness page appears once Garmin has sent a weight. Weight can also fill the Diary's Body Stats weight when **Settings > Wellness > Mirror wellness weight to Body Stats** is on.

Not read: workouts and GPS routes, VO₂ max, training readiness, women's health, and the food log. Garmin keeps VO₂ max and recovery numbers out of the Personal API.

## How syncing works

- **When**: the same **Sync Mode** as the partner setup: on opening the Wellness page, manual only, or **Scheduled** at an interval or time of day. **Sync** on the card or the Wellness page pulls right away.
- **How far back**: 30 days on the first sync and after a new key. After that, the last 3 days (so a watch that uploads late still lands), back to the last good sync after a gap of up to 30 days, or the **Sync Range** you set when you tap Sync.
- **Gentle on Garmin**: each sync is four requests that each cover a date range (daily summaries, sleep, HRV, weight), split only where Garmin limits the range. Two syncs at once share one pull, and automatic syncs wait at least 30 seconds between pulls. If Garmin says it's limiting requests, the sync stops and tries again next time.
- **Tokens**: the server swaps the key for an access token that lasts about a day, keeps it until shortly before it ends, and gets a new one when Garmin asks. Every request has a time limit, so a slow Garmin never holds up the server.

## When the key expires

Garmin keys last 30 days to a year and can't be renewed: you create a new one.

- If you entered the expiry date, Settings shows how many days are left, and a week before it ends you get a notice through your push service when **Settings > Notifications > Sync Failures** is on. Garmin also emails you about a week before.
- When Garmin stops accepting the key, NutriTrace stops trying, the card shows **Needs a New Key** with the steps again, and you get one notice. Readings already synced stay.
- To keep syncing, create a new key in Garmin Connect (Garmin asks you to delete the old one first) and select **Replace Key**, or paste it into the form the card shows. The new key gets the 30-day backfill, so the days in between are filled in.

## Privacy and storage

- The key and its access token are stored encrypted on your server (AES-GCM, with `TOKEN_ENC_KEY` or a key derived from `JWT_SECRET`), one row per account. NutriTrace never shows the key again, never logs it, and never sends it to the browser or the Android app.
- Full backups leave the key out, like every other wearable token: after restoring onto another server, paste the key again. Rotating `JWT_SECRET` without `TOKEN_ENC_KEY` makes the stored key unreadable; the card then asks for the key again.
- **Disconnect** deletes the stored key and token. Deleting an account deletes them too.
- The server only talks to `personal.connectapi.garmin.com` (or `personal.connectapi.garmin.cn` for China accounts).

## Android app

The Android app uses the Personal API Key through your server, so in server mode it shows the same card and the same data. In local mode (no server) Garmin isn't available; use [Health Connect](health-connect.md) to read what Garmin Connect writes on the phone.

## Health API partner setup

If your organization has an approved Garmin Health API partnership: in **Settings > Wellness > Garmin**, choose **Health API Partner**, paste the **Consumer Key** and **Consumer Secret**, register `https://your-domain/api/wellness/garmin/callback` as the redirect URI in your Garmin app, then **Save & Connect**. This path needs a public HTTPS address. When both are set up, the Personal API Key is the one NutriTrace syncs with.

## Related

- [Wellness and wearables](wellness.md)
- [Health Connect (Android)](health-connect.md)
- [Goals & Adaptive TDEE](goals.md)
