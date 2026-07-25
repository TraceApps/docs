# Apprise

[Apprise](https://github.com/caronc/apprise) is a universal notification library that speaks Discord, Telegram, Matrix, Slack, Pushover, SMTP, and roughly seventy other targets through a single HTTP API. TraceApps posts to Apprise once; Apprise fans the message out to every destination you configured. Handy when you want the same notification in Discord and email and your ntfy topic without wiring three services into the app.

## Setup

**1. Run Apprise.** The maintained image is `caronc/apprise`. Minimal compose:

```yaml
services:
  apprise:
    image: caronc/apprise:latest
    restart: unless-stopped
    volumes:
      - ./apprise/config:/config
    ports:
      - "8000:8000"
```

**2. Give Apprise its targets.** Drop a `config.yml` into `./apprise/config/` naming each destination. One tag can group several URLs so a single POST fans out to all of them:

```yaml
urls:
  - discord://webhook_id/webhook_token:
      - tag: cooktrace
  - mailto://user:pass@smtp.example.com:
      - tag: cooktrace
  - ntfys://ntfy.example.com/nutritrace:
      - tag: nutritrace
```

The full URL grammar is on the [Apprise wiki](https://github.com/caronc/apprise/wiki). Reload Apprise (`docker compose restart apprise`) after editing.

**3. Point TraceApps at it.** In the app, open **Settings, Notifications**:

- Set **Push service** to `Apprise`.
- **Apprise Server URL**: base URL of your Apprise instance, e.g. `http://apprise:8000` if it is on the same docker network, or `https://apprise.example.com` if it sits behind your reverse proxy. The app appends `/notify` internally, so do not include a path.
- **Tag** (optional): the tag you defined in `config.yml`. Leave blank to send to all configured URLs.

The stored setting keys are `notifPushService=apprise`, `appriseUrl`, and `appriseTag`. Values live per user in `user_settings` and never leave the server after storage; the browser sends `{ title, message, priority }` to `POST /api/notify` and the server posts the outbound call.

**4. Test.** Hit the "Send test" button next to the dropdown. A `Test notification, push service connected` message should land on every configured Apprise target within a second.

## Priority mapping

TraceApps sends a numeric priority (1 to 8). Apprise doesn't have a native priority field, so the server maps priority `>= 7` to Apprise `type: warning` and everything else to `type: info`. Individual notification services then map `warning` to whatever they support (Discord embed colour, Pushover priority, etc).

## Related

- [Push notifications overview](notifications.md)
- [Gotify](gotify.md)
- [ntfy](ntfy.md)
