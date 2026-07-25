# Gotify

[Gotify](https://gotify.net/) is a self-hosted push server with first-party Android and web clients. Small, single-binary, no external dependencies. TraceApps posts messages to Gotify's HTTP API; the Gotify Android app receives them over a long-lived websocket, which means delivery works even when the TraceApps app is closed.

Good pick if you want a dedicated push app on your phone that you control end-to-end.

## Setup

**1. Run Gotify.** Official image:

```yaml
services:
  gotify:
    image: gotify/server:latest
    restart: unless-stopped
    environment:
      GOTIFY_DEFAULTUSER_PASS: change-me
    volumes:
      - ./gotify/data:/app/data
    ports:
      - "8080:80"
```

Open the web UI, log in (`admin` / the password you set), and change the password immediately.

**2. Create an Application.** In the Gotify UI: **Apps, Create Application**. Give it a name like `CookTrace` or `NutriTrace`. Gotify prints an application token; copy it. Each application gets its own icon and its own message stream, so create one per TraceApps install if you host several.

**3. Point TraceApps at it.** Open **Settings, Notifications**:

- Set **Push service** to `Gotify`.
- **Gotify Server URL**: base URL of your Gotify instance, e.g. `https://gotify.example.com`. The app appends `/message?token=...` internally.
- **App token**: the token you copied.

Stored setting keys: `notifPushService=gotify`, `gotifyUrl`, `gotifyToken`. The token stays on the server, never returned to the browser after storage.

**4. Install the Gotify mobile app.** The Android client is on [F-Droid](https://f-droid.org/packages/com.github.gotify/) and [Google Play](https://play.google.com/store/apps/details?id=com.github.gotify). Log in with your Gotify server URL and user credentials. Notifications from any Application you have visibility on will arrive. A desktop tray client is also available for Windows, macOS, and Linux.

**5. Test.** Hit the "Send test" button in TraceApps. The Gotify app should ping within a second.

## Priority mapping

TraceApps sends priorities 1 to 8; Gotify accepts the same range and uses the value to decide the notification channel on Android (silent, default, high). Wellness alerts and sync-failure notifications fire at priority 7 to 8 so they bypass Do Not Disturb on most Android setups; regular reminders sit at 4 to 5.

## Related

- [Push notifications overview](notifications.md)
- [Apprise](apprise.md)
- [ntfy](ntfy.md)
