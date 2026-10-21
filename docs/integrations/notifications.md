# Push notifications overview

Every Trace app has the same two-channel notification model. You can turn either channel on independently, or run both at once for belt-and-braces delivery.

## Two channels

### 1. Device notifications

The one users expect. On the PWA, Trace uses the browser's Web Notification API; on the Android app, Capacitor `LocalNotifications` (or, for NutriTrace, native Android WorkManager). Notifications land on the device that opened the app.

Turned on per-user via `Settings → Notifications → Enable device notifications` (setting key `notifLocalEnabled`). The browser or OS prompts for permission the first time; after that, delivery is silent.

Suits a solo self-hoster on a laptop or phone. Doesn't help if you want a notification while the app isn't open on any device, or if you want to fan out to a Home Assistant / desktop shell / smartwatch that isn't your primary device.

### 2. Push service

Server-side delivery via one of three OSS push relays. The Trace server calls the relay's HTTP API; the relay does the last-mile push to your phone, desktop, Home Assistant, whatever else you've wired.

One service at a time, exclusive: `Settings → Notifications → Push service` is a single dropdown with `None / Apprise / Gotify / ntfy`. Picking one reveals its config fields. The API URL and any secrets are stored per user (`user_settings`) and never sent to the browser after storage; the client posts only `{ title, message, priority }` to `POST /api/notify`, and the server-side `push-notify.js` module does the outbound call.

Suits multi-device setups, always-on delivery, integrating with existing self-hosted infra.

Both channels can be on simultaneously. Trace has a shared dedup key format between the server push and the client-side reminder scheduler, so you don't get two of the same notification when both fire.

### From the Android app

In LiftTrace (from its next release), an Android app signed in to a server with user accounts asks your server to send each push, as the web app does, from the settings your account keeps. A push service on your home network (`http://apprise:8000`, `http://192.168.1.20:8080`) works that way. Otherwise (Use Locally, a server without user accounts, or a server it can't reach just then), the phone sends the push itself with what you set in Settings > Notifications, ntfy's access token and Apprise's tag included. Earlier versions of the app never sent Workout Complete or PR pushes from the phone at all.

A release build of an Android app only reaches `https://` addresses itself, so when the phone sends a push, the push service needs an HTTPS address. See [HTTPS on the LAN](../self-hosting/lan-https.md).

## Per-app reminder catalog

The reminder toggles under `Settings → Notifications` differ per app because the domain differs.

### CookTrace

- **Expiration digest**: pantry items nearing their expiration date. Daily. Configurable window (`notifExpiryDaysBefore`) and delivery time (`notifExpiryTime`).
- **Cook-day reminder**: a cook you've planned for today.
- **Thaw alert**: reminder to move something from freezer to fridge.
- **Shopping-list nudge**: standing shopping-list items you haven't cleared.
- **Recipe comments**: someone commented on a shared recipe.
- **Weekly summary**: cooks this week, top dish, new recipes, next week's plan, pantry/shopping snapshot. Email, not push.
- **Backup failed**: scheduled auto-backup did not complete. Admin-only.

### LiftTrace

- **Rest-timer**: the between-sets timer, delivered as a device notification so you can lock your phone between sets.
- **Workout reminder**: scheduled workout not yet started.
- **Rest-day reminder**: rest day is today, not a workout day.
- **Streak alert**: your training streak is at risk.
- **Workout complete**: end-of-session summary.
- **PR celebration**: a new personal record was hit.
- **Coach feedback**: a coach left feedback on your session.
- **Weekly summary**: volume, workouts done, PRs.

### NutriTrace

- **Water reminders**: hourly / custom interval prompts to log water.
- **Meal reminders**: breakfast / lunch / dinner / snack windows.
- **Goal celebrations**: you hit a calorie / macro / step goal today.
- **Step goal**: end-of-day step progress.
- **Weigh-in reminder**: scheduled weigh-in day.
- **Bedtime + wind-down**: two-stage evening nudge.
- **Fasting complete**: your intermittent fast window closed.
- **Weekly summary**: average calories, protein, steps, sleep, goal hit rate and weight change, once a week on your **Delivery Day** at your **Delivery Time**. It goes to each channel you have: by push through your push service, by email when the server sends email and your account has an address, or both. With email alone, Settings > Notifications still shows it, and its description says how it will arrive (from NutriTrace's next release; earlier versions sent the email only to people with a push service set up).
- **Wellness alerts**: readiness / stress / sleep anomalies.
- **Workout summary**: post-workout recap.
- **Sync failures**: Fitbit / Google Health / Withings sync stopped working.

!!! note "NutriTrace on Android uses WorkManager"
    NutriTrace's Android build routes reminders through native Android WorkManager rather than the JS `LocalNotifications` layer. The `_USE_NATIVE_WORKER` switch in `src/lib/notifications.js` gates this. Reminder timing survives app kill and low-power modes far better than the JS path, but the trade-off is that the JS reminder scheduler is a no-op on NutriTrace-Android. CookTrace and LiftTrace still use the JS path on Android; migration to WorkManager is on the roadmap but not yet in.

## Provider setup pages

Each push service has its own setup page with URL formats, token generation, and a working example:

- [Apprise](apprise.md)
- [Gotify](gotify.md)
- [ntfy](ntfy.md)

These are P1 stubs at the moment; the fields under `Settings → Notifications` are self-explanatory in the meantime (`URL` plus `Token`, or `URL` plus `Topic` plus optional `Token` for ntfy).

## Related

- [Email / SMTP](smtp.md)
