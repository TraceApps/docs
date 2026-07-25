# Fitbit → Google Health migration

Google is retiring the legacy Fitbit Web API. New Fitbit app registrations at `dev.fitbit.com` are already refused; existing apps stop working when Google flips the switch. Fitbit data now flows through the **Google Health API**, and NutriTrace has followed suit: existing Fitbit connections must be re-linked via Google Cloud OAuth before the deadline below.

!!! danger "Deadline: 2026-05-31"
    NutriTrace removes its legacy Fitbit Web API code path entirely on **May 31, 2026**, roughly four months ahead of Google's own cutoff. If your Fitbit was connected to NutriTrace before the Google Health migration was released, **re-link via the Google Cloud setup below by May 31** to avoid losing wellness data flow. Settings → Wellness surfaces a re-link banner with a jump-off to start the process. After May 31 the legacy path is gone and new setup starts from scratch on the Google Health page.

The Fitbit account, devices, historical data on Fitbit's servers, and the Fitbit mobile app itself are unchanged. Only the pipe NutriTrace uses to read your data has changed.

## Prerequisite: migrate your Fitbit account to Google

Google Health OAuth returns 403 for accounts that have not been switched over. In the Fitbit mobile app, follow the **Continue with Google** prompt to merge your Fitbit account into Google. Once done, your Fitbit login is your Google account. This has to happen before any of the setup below will work.

## Set up Google Cloud OAuth

You need a **Client ID** and a **Client Secret** from Google Cloud. Both go into **Settings → Wellness → Fitbit** on NutriTrace.

### 1. Enable the Health API

Open [console.developers.google.com/apis/library/health.googleapis.com](https://console.developers.google.com/apis/library/health.googleapis.com) and click **Enable**. If you don't have a Cloud project yet, Google will prompt you to create one; a personal project with the default name is fine.

### 2. Create OAuth credentials

Open [console.developers.google.com/apis/credentials](https://console.developers.google.com/apis/credentials), then **Create Credentials → OAuth client ID**.

- **Application type**: **Web server**.
- **Authorized redirect URI**: `https://your-nutritrace-domain.com/api/wellness/google-health/callback` (substitute your actual public HTTPS origin; if you use a `BASE_URL` subpath prefix, include it: `https://your-domain.com/nutritrace/api/wellness/google-health/callback`).
- **Name**: whatever you like ("NutriTrace" works).

Click **Create**. Google shows a **Client ID** and **Client Secret**. Copy both.

### 3. OAuth consent screen (may be prompted)

If Google's console prompts you to configure an OAuth consent screen before letting you finish, choose **External / Testing** and add yourself as a test user. Most single-user setups won't see this step; Google's flow handles it implicitly during credential creation. Test mode is capped at 100 users, which is more than enough for a family instance. For larger deployments you would go through Google's verification process to publish to production.

### 4. Paste into NutriTrace

**Settings → Wellness → Fitbit** on your NutriTrace instance. Paste the Client ID and Client Secret, save, then click **Connect**. Google's consent screen opens, grant the requested scopes, and the callback lands you back on the Wellness page with the connection banner turned green.

Setup walkthrough on Google's side: <https://developers.google.com/health/setup>.

!!! warning "Public HTTPS required"
    The callback URL must be a real public HTTPS origin. `localhost`, `192.168.x.x`, and LAN-only hostnames are refused by Google's OAuth. If your NutriTrace runs on a LAN with no public origin, you have four supported paths documented under [HTTPS on the LAN](../self-hosting/lan-https.md). The most common resolution is a Cloudflare Tunnel or Tailscale funnel pointing at your NutriTrace container.

## What the connection endpoints look like

Configured per-user under `wellness_config`; there are no env vars for these credentials, since each user typically has their own Fitbit account and Google project. The relevant endpoints (`server/routes/google-health.js`):

- `GET /api/wellness/google-health/config` / `PUT`: read / write the client id + secret pair.
- `GET /api/wellness/google-health/status`: connection state, last sync, last error.
- `GET /api/wellness/google-health/authorize`: begin OAuth flow.
- `GET /api/wellness/google-health/callback`: Google redirects back here.
- `POST /api/wellness/google-health/sync`: force a sync now.
- `POST /api/wellness/google-health/disconnect`: revoke and forget.

Two diagnostics also live there:

- `GET /api/wellness/google-health/smoke-steps`: minimal round-trip smoke test for the connection.
- `GET /api/wellness/google-health/test-all`: end-to-end sync verification across every metric.

## Legacy `fitbit.js` parallel until September 2026

The legacy Fitbit Web API code path in `server/routes/fitbit.js` still exists on disk. It stays parallel to the new Google Health path until **September 2026**, at which point the file is removed. Between now and then, the two paths can coexist but the app prefers Google Health when a user has both configured. This staggered removal is deliberate: it gives you a window where old Fitbit tokens keep working while you complete the Google Cloud setup at your own pace.

If you re-linked recently and want to be sure the old path is dormant, `wellness_config` for your user will show `google_health` populated and the Wellness page will surface the Google Health banner in place of the old Fitbit one.

## USER_PREFS vs DEVICE_PREFS

The Client ID, Client Secret, and connection state are **user-scoped** and live on the server (they are filtered from any sync payload that would leave the server per `server/lib/server-only-keys.js`). The Wellness page toggles for which metric tiles to render are `USER_PREFS` and sync across devices. Nothing Google-Health-related is per-device.

## Related

- [Wellness & wearables](wellness.md)
- [Withings](withings.md)
- [Goals & Adaptive TDEE](goals.md)
- [HTTPS on the LAN](../self-hosting/lan-https.md)
