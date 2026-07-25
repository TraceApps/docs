# ntfy

[ntfy](https://ntfy.sh/) is a dead-simple HTTP-based pub/sub push service. Pick a topic name, POST to `https://ntfy.sh/<topic>`, and every subscriber to that topic gets the message. The mobile app is on F-Droid and Play; the web app works from any browser. No accounts required for the public instance, though self-hosting is straightforward.

The lowest-friction of the three push options: you can be sending notifications inside two minutes without running anything yourself.

## Two ways to run it

### Public `ntfy.sh` instance

Free, no account, works out of the box. Fine for personal use and small setups. The public instance rate-limits requests (roughly 60 messages per hour and a handful of concurrent subscribers per topic without an account); paid tiers raise the limits. Anyone who guesses your topic name can send to it, so pick something unguessable if that matters to you.

### Self-hosted

For unlimited throughput, ACL-controlled topics, or offline use.

```yaml
services:
  ntfy:
    image: binwiederhier/ntfy
    command: serve
    restart: unless-stopped
    volumes:
      - ./ntfy/cache:/var/cache/ntfy
      - ./ntfy/config:/etc/ntfy
    environment:
      NTFY_BASE_URL: https://ntfy.example.com
      NTFY_CACHE_FILE: /var/cache/ntfy/cache.db
      NTFY_AUTH_FILE: /var/cache/ntfy/auth.db
      NTFY_BEHIND_PROXY: "true"
    ports:
      - "8080:80"
```

Front it with Caddy or nginx for TLS. The [ntfy config docs](https://docs.ntfy.sh/config/) cover ACL setup, user accounts, and web-push subscription.

## Point TraceApps at it

Open **Settings, Notifications**:

- Set **Push service** to `ntfy`.
- **ntfy Server URL**: `https://ntfy.sh` (the default) or your own base URL, e.g. `https://ntfy.example.com`.
- **Topic**: any string. Choose something unguessable if you're on the public instance, e.g. `nutritrace-4f2a9c8b1d`.
- **Token** (optional): a Bearer token, needed only if the server or the topic requires auth. Blank is fine for the public instance and for open topics on a self-hosted server.

Stored setting keys: `notifPushService=ntfy`, `ntfyUrl`, `ntfyTopic`, `ntfyToken`. The token stays server-side.

## Subscribe on your device

Install the [ntfy Android app](https://f-droid.org/en/packages/io.heckel.ntfy/) or [iOS app](https://apps.apple.com/us/app/ntfy/id1625396347), tap **Subscribe to topic**, enter your server URL and topic name. Notifications arrive over a battery-friendly long-polling connection.

Multiple people can subscribe to the same topic; every subscriber gets every message. That is how you fan a household's notifications to several phones without configuring anything server-side.

## Priority mapping

TraceApps sends priorities 1 to 8. ntfy accepts 1 to 5, so the server clamps the value with `Math.min(5, priority)`. Wellness alerts and sync failures land at ntfy priority 5 (max, "urgent"), regular reminders at 4 or 5.

## Related

- [Push notifications overview](notifications.md)
- [Apprise](apprise.md)
- [Gotify](gotify.md)
