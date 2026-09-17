# Webhooks

NoteTrace can send a signed HTTP POST when something happens, for wiring notes into n8n, Home Assistant, Node-RED, or your own scripts without polling. Webhooks are off by default.

## Enable

```env
WEBHOOKS_ENABLED=1
# Only if the target is on a private or loopback address (a Home Assistant
# container on the same Docker network, for example):
ALLOW_PRIVATE_WEBHOOK_URLS=1
```

Then add a webhook under **Settings, Webhooks** (admin, multi-user mode). Enter the target URL and choose events. A signing secret is generated (or supply your own) and shown once; save it to verify deliveries. **Send test event** checks the target without waiting for a real event.

## Events

| Event | Fires when | `data` |
|---|---|---|
| `note.created` | A note is created from any device, including through sync | `{ "note_id", "title", "kind" }` |
| `checklist.completed` | The last unchecked item on a checklist is checked | `{ "note_id", "title" }` |
| `reminder.fired` | A reminder occurrence comes due (once per occurrence of a repeat) | `{ "note_id", "title", "reminder_at", "repeat" }` |

Imports don't fire `note.created`, so a large Google Keep import doesn't flood the receiver. Events go to webhooks owned by the note's owner.

`reminder.fired` is sent by the server's reminder check, which runs every minute. `reminder_at` is the occurrence time in ISO 8601 UTC and `repeat` is `daily`, `weekly`, `monthly`, `yearly`, or `null`. It fires whether or not a push service is set up, which makes it a good trigger for smart-home announcements.

## Payload and signature

```json
{
  "event": "reminder.fired",
  "timestamp": "2026-09-14T12:00:04.211Z",
  "data": { "note_id": 42, "title": "Water the plants", "reminder_at": "2026-09-14T12:00:00.000Z", "repeat": "weekly" }
}
```

Headers:

```
X-NoteTrace-Signature: sha256=<hex HMAC-SHA256 of the raw body with your secret>
X-NoteTrace-Event: reminder.fired
X-NoteTrace-Delivery: <unique id per delivery attempt>
```

Verify by computing the HMAC over the exact bytes received:

```js
import { createHmac, timingSafeEqual } from 'node:crypto';

function verify(rawBody, header, secret) {
  const expected = createHmac('sha256', secret).update(rawBody).digest('hex');
  return timingSafeEqual(Buffer.from(expected), Buffer.from(header.replace('sha256=', '')));
}
```

## Retries

A delivery that errors, times out, or gets a non-2xx response is tried up to 3 times with a short backoff. There's no long-term queue: if the receiver is down for more than a few seconds, that event isn't sent later. The last result for each webhook shows in Settings.
