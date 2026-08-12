# Federation API (v1)

The versioned API NutriTrace exposes to sibling apps (CookTrace, LiftTrace) and any other authorized integration. Base path `/api/v1/`. Wire format is JSON both directions.

For the cross-app story and how tokens fit into it, see [Federation (cross-app links)](../integrations/federation.md).

## Auth

Every `/api/v1/*` request needs a Bearer token:

```
Authorization: Bearer nt_pat_<43 chars>
```

Tokens are personal access tokens minted in Settings > **API Tokens** (admin). The raw value has an `nt_pat_` prefix so leaked tokens are easy to spot in logs and credential scanners; NT hashes with SHA-256 at rest and shows you the raw value exactly once on creation.

Errors carry a stable `code` field:

- `401 auth_missing`. No Bearer header.
- `401 auth_invalid`. Unknown or revoked token.
- `403 auth_scope`. Token exists but lacks the required scope.
- `429 rate_limited`. Over the per-token rate cap; check `Retry-After`.

Every response includes `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (unix seconds) so clients can back off gracefully.

## Scopes

Two scopes today, gated per-endpoint. Tokens can hold any combination.

| Scope | Grants | Used by |
|---|---|---|
| `read:foods` | Read the token owner's foods library | CookTrace |
| `write:workouts` | Post workouts into the token owner's wellness history (Settings → Wellness → Workout History) | LiftTrace |
| `write:activity` | Log manual activity entries into the diary Activity section | External trackers, headless integrations |
| `write:body-measurements` | Push scale readings into the diary's body-stats | Home Assistant, Node-RED, Gadgetbridge |

Adding unknown scopes at token-creation time is silently dropped (forward-compat for clients written against a future NT). A token with no valid scopes is rejected.

## Endpoints

### `GET /api/v1/me`

Identity check. Always available regardless of token scopes; use it to verify a token is valid and discover which user + instance it belongs to.

```bash
curl -H "Authorization: Bearer $NT_TOKEN" \
  https://nutritrace.example.com/api/v1/me
```

Returns:

```json
{
  "user": { "id": 1, "username": "alice", "full_name": "Alice", "role": "admin" },
  "instance": { "url": "https://nutritrace.example.com", "version": "1.0.3" },
  "scopes": ["read:foods", "write:workouts"]
}
```

### `GET /api/v1/foods`

List foods in the token owner's library. Requires `read:foods`.

Query params:

- `q`. Case-insensitive substring match against name and brand.
- `category`. Exact-match category filter.
- `limit`. 1..500, default 100.
- `offset`. Pagination cursor.

```bash
curl -H "Authorization: Bearer $NT_TOKEN" \
  "https://nutritrace.example.com/api/v1/foods?q=chickpea&limit=25"
```

Returns:

```json
{
  "items": [
    {
      "id": 42,
      "name": "Chickpeas, canned",
      "brand": "Store brand",
      "category": "Legumes",
      "barcode": "0123456789012",
      "portion": 100,
      "unit": "g",
      "img_url": "/uploads/foods/42.jpg",
      "notes": null,
      "nutrition": { "kcal": 139, "protein_g": 7.5, "carbs_g": 22, "fat_g": 2.3 },
      "created_at": "2026-01-14T09:22:11Z",
      "updated_at": "2026-03-02T18:04:03Z"
    }
  ],
  "total": 3,
  "limit": 25,
  "offset": 0
}
```

`img_url` may be relative to the NT origin; the client is responsible for resolving it if it needs to fetch the image.

### `GET /api/v1/foods/:id`

Single food by numeric ID. Requires `read:foods`. Returns the same shape as one array element from the list endpoint, or `404 not_found` if the ID is unknown or belongs to a different user.

```bash
curl -H "Authorization: Bearer $NT_TOKEN" \
  https://nutritrace.example.com/api/v1/foods/42
```

### `POST /api/v1/workouts`

Log a completed workout into the token owner's wellness history. Requires `write:workouts`.

Body:

- `date` (required, `YYYY-MM-DD`).
- `calories_burned` (required, 0..10000).
- `external_id` (required, up to 128 chars). Idempotency key; re-posting the same ID updates the existing row instead of duplicating.
- `name` (optional, free text, defaults to `Workout`).
- `duration_min` (optional, 0..1440).
- `start_time` (optional, ISO 8601 string).

```bash
curl -X POST \
  -H "Authorization: Bearer $NT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "date": "2026-07-25",
    "name": "Push day",
    "duration_min": 58,
    "calories_burned": 340,
    "start_time": "2026-07-25T07:12:00Z",
    "external_id": "lt:workout:12345"
  }' \
  https://nutritrace.example.com/api/v1/workouts
```

Returns:

```json
{
  "ok": true,
  "workout_id": 87,
  "date": "2026-07-25",
  "calories_burned": 340,
  "daily_total_calories_burned": 340
}
```

NT stores the row as `source='lifttrace'` and rolls the day's total up into `wellness_data` under `metric_type='calories_out'` so the cross-source `/api/wellness/calories-out` lookup can find it. The wearable-vs-federation priority is enforced by that lookup, not by the write, so posting is always safe: NT figures out how to combine sources at read time.

Not to be confused with `POST /api/v1/activity` (below), which targets the diary's Activity section instead of the Workout History under Wellness.

### `POST /api/v1/activity`

Log a manual activity entry into the diary's Activity section. Requires `write:activity`.

Use this for external integrations that should show up as diary activities (a Dropbox / TCX / Node-RED pipeline pushing cardio from a map app, a headless HA rule attributing calories, etc.). Writes to the same `activity_log` table the in-app "Add Activity" sheet uses, so entries render in the diary the same way and contribute to the day's kcal-burned tally.

Body:

- `date` (required, `YYYY-MM-DD`).
- `name` (required, free text, up to 80 chars).
- `kcal` (required, 0..10000).
- `duration_min` (optional, 0..1440).
- `distance` (optional, free text, up to 40 chars — the diary treats it as a display-only string).
- `external_id` (optional, up to 128 chars). Idempotency key; re-posting the same ID updates the existing entry instead of duplicating.

```bash
curl -X POST \
  -H "Authorization: Bearer $NT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "date": "2026-08-12",
    "name": "Bicycle",
    "duration_min": 40,
    "kcal": 270,
    "distance": "12.4 km",
    "external_id": "external:workout:abc-123"
  }' \
  https://nutritrace.example.com/api/v1/activity
```

Returns:

```json
{
  "ok": true,
  "activity_id": 42,
  "date": "2026-08-12",
  "kcal": 270,
  "daily_total_kcal": 540,
  "updated": false
}
```

`updated: true` means the row was matched by `external_id` and updated in place; `false` means a new entry was inserted.

## Rate limits

Per-token sliding window, `API_RATE_LIMIT_PER_MIN` (default 60) requests per minute. On exceed the endpoint returns `429 rate_limited` with a `Retry-After` header (seconds). Bump the env var if you have a chatty integration.

## Related

- [Federation (cross-app links)](../integrations/federation.md)
- [NutriTrace federation (CookTrace side)](../cooktrace/nt-federation.md)
- [NutriTrace federation (LiftTrace side)](../lifttrace/nt-federation.md)
