# Webhooks

CookTrace can send a signed HTTP POST when something happens, for wiring your kitchen into n8n, Home Assistant, Node-RED, or your own scripts without polling. It's the push side of the [public REST API](public-api.md). Webhooks are off by default.

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
| `meal.cooked` | A recipe is logged as cooked: a new cooked entry in the Cook Diary, marking a planned meal as cooked, **Mark as Cooked** on a recipe, the REST API, or the MCP `log_cook` tool | `{ "date", "recipe_id", "recipe_name", "kind", "servings", "rating", "meal_type" }` |
| `shopping_list.completed` | The last unchecked item on the shopping list is checked | `{ "items_count" }` |
| `pantry.out_of_stock` | A pantry item that was in stock is marked out of stock | `{ "pantry_item_id", "name" }` |

Events go to webhooks owned by the account that made the change. Re-checking an item that was already checked doesn't fire `shopping_list.completed` again.

!!! note "Android app"
    Changes made in the Android app reach the server through sync, and sync doesn't fire webhooks yet. Cooking, finishing the list, or running out of something on the phone won't send an event; the same action in the web app, the REST API, or MCP will.

## Target URLs

A target must be `http` or `https`. Link-local and cloud-metadata addresses (such as `169.254.169.254`) are always refused. Private and loopback addresses are refused unless `ALLOW_PRIVATE_WEBHOOK_URLS=1` is set. Every address the host resolves to is checked, and the target is checked again right before each delivery, not only when you save it.

## Payload and signature

```json
{
  "event": "meal.cooked",
  "timestamp": "2026-09-19T18:30:04.211Z",
  "data": {
    "date": "2026-09-19",
    "recipe_id": 42,
    "recipe_name": "Weeknight Chili",
    "kind": "cooked",
    "servings": 4,
    "rating": 5,
    "meal_type": "dinner"
  }
}
```

Headers:

```
X-CookTrace-Signature: sha256=<hex HMAC-SHA256 of the raw body with your secret>
X-CookTrace-Event: meal.cooked
X-CookTrace-Delivery: <unique id per delivery attempt>
```

Verify by computing the HMAC over the exact bytes received, not a re-serialized copy:

```js
import { createHmac, timingSafeEqual } from 'node:crypto';

function verify(rawBody, header, secret) {
  const expected = createHmac('sha256', secret).update(rawBody).digest('hex');
  return timingSafeEqual(Buffer.from(expected), Buffer.from(header.replace('sha256=', '')));
}
```

## Retries

A delivery that errors, times out, or gets a non-2xx response is tried up to 3 times with a short backoff. There's no long-term queue: if the receiver is down for more than a few seconds, that event isn't sent later. The last result for each webhook shows in Settings.
