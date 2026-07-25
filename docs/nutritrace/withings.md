# Withings

Withings makes smart scales (Body, Body+, Body Cardio, Body Scan, Body Comp), plus sleep mats, watches, and blood pressure cuffs. NutriTrace's Withings integration pulls **weight and body-composition** data, which feeds the Adaptive TDEE weight input, the Body Stats card on the Diary, and any weight-based charts on Statistics.

Setup is per-user, OAuth 2.0, and needs a free developer account on Withings' side. Unlike Fitbit / Google Health, there is no company-wide cutoff deadline; the integration is stable.

## Prerequisite: developer app on Withings

Open [developer.withings.com](https://developer.withings.com), sign in with your Withings account, and register a **Public API Integration**:

1. Choose **Public Cloud** (the free tier).
2. **Application name**: anything (`NutriTrace` works).
3. **Description**: short. Not shown to end users of your own instance.
4. **Callback URI**: `https://your-nutritrace-domain.com/api/wellness/withings/callback`. If you mount NutriTrace under a subpath with `BASE_URL`, include the prefix: `https://your-domain.com/nutritrace/api/wellness/withings/callback`.
5. Save.

Withings shows a **Client ID** and **Consumer Secret** on the app detail page. Copy both.

!!! warning "Public HTTPS required"
    Withings OAuth refuses callback URLs on `localhost`, `192.168.x.x`, or LAN-only hostnames. If your NutriTrace has no public HTTPS origin, use one of the paths under [HTTPS on the LAN](../self-hosting/lan-https.md) (Cloudflare Tunnel, Tailscale funnel, self-signed CA, or a real public domain).

## Configure NutriTrace

**Settings → Wellness → Withings**:

1. Paste the **Client ID** and **Consumer Secret**.
2. Save. The **Connect** button becomes enabled.
3. Click **Connect**. Withings' consent screen opens; grant the requested scopes (`user.info`, `user.metrics`, `user.activity`).
4. The callback lands you back on the Wellness page with a **green** connection banner.

Force a sync from the same banner (the ⟳ button) or wait for the scheduled sync. Sync mode, interval, and window follow the same shape as the other wearables (`withingsSyncMode`, `withingsSyncInterval`, `withingsSyncWindowStart`, `withingsSyncWindowEnd`).

## Sync cadence

Withings tokens are refreshed automatically; you should not have to re-authorize under normal operation. Sync runs:

- **On login** (once per session, when the app opens fresh).
- **On the configured interval** while the app is open (default: hourly).
- **On demand** via the ⟳ button on the Wellness card.

The Withings API returns readings incrementally, so a routine sync pulls only what is new since the last successful call. Historical backfill on first connect walks the range you set under `withingsSyncRange` (default: 30 days).

## What NutriTrace pulls

From the `user.metrics` scope, primarily:

- **Weight** (`weight_kg`).
- **Body composition**: fat mass (%), muscle mass (%), bone mass (%), body water (%), lean/fat mass, visceral fat, vascular age, metabolic age, BMR. Coverage depends on your scale model; a basic Body scale gives weight + fat %, a Body Scan gives the full lineup.
- **Heart-rate readings** captured by the scale during weigh-in, when your model measures them.

These land in the `wellness_data` table with `source='withings'`. The cross-source `/api/wellness/latest?metric=weight_kg` prefers Withings when a fresh reading exists (see [Wellness and wearables](wellness.md) for the priority order).

## Endpoints (reference)

Handled in `server/routes/withings.js`. Per-user credentials, no env vars.

- `GET /api/wellness/withings/config`, `PUT`: read / write client id + secret.
- `GET /api/wellness/withings/status`: connection state, last sync, last error.
- `GET /api/wellness/withings/authorize`: begin OAuth flow.
- `GET /api/wellness/withings/callback`: Withings redirects back here.
- `POST /api/wellness/withings/sync`: force a sync now.
- `POST /api/wellness/withings/disconnect`: revoke and forget.

## USER_PREFS vs DEVICE_PREFS

Client ID, secret, and connection state are **user-scoped** on the server and never leave it (`server/lib/server-only-keys.js`). The sync-mode / interval / window toggles are `USER_PREFS` and travel with the account across devices. Nothing Withings-specific is per-device.

## Related

- [Wellness and wearables](wellness.md)
- [Fitbit and Google Health migration](google-health.md)
- [Goals and Adaptive TDEE](goals.md)
