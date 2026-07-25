# Troubleshooting

A running list of the failure modes that actually catch self-hosters out. Organised as a Q&A. If your symptom is not here, turn on `LOG_LEVEL=debug` in the container and check the server logs first; verbose logs surface most of these cases explicitly.

## Login and cookies

### Login succeeds but every request 401s over plain HTTP { #login-401-loop }

**Symptom:** The login form accepts your password, the browser redirects, and then every subsequent request returns 401. Refreshing sends you back to the login page.

**Cause:** Auth cookies are marked `Secure` by default. Over plain `http://` the browser silently drops them, so the session never actually sticks. Firefox logs the rejection in its console; Safari fails without any warning at all.

**Fix:** Add `INSECURE_COOKIES=1` to the container environment and restart. Only do this on a trusted LAN; the variable is named the way it is for a reason. The better long-term answer is real TLS. See [HTTPS on the LAN](../self-hosting/lan-https.md) for how to set that up with Let's Encrypt, a Cloudflare Tunnel, or a self-signed CA installed on your devices.

### Android release APK will not connect to my LAN server

**Symptom:** The web PWA loads fine from your laptop, but the release-signed Android APK refuses to connect. The error looks like a network timeout or an SSL handshake failure.

**Cause:** Android's release-build network security config bans cleartext (`http://`) traffic outright. This is enforced by the OS, not the app.

**Fix:** Use HTTPS. The four supported paths:

- Real domain plus Let's Encrypt (via Caddy, Traefik, or nginx with certbot).
- Cloudflare Tunnel or Tailscale Funnel (both give you a public cert without opening ports).
- A self-signed CA installed on the Android device's trust store.
- Build the debug APK yourself with `npm run android:debug` in the app repo; the debug variant is allowed to talk to `http://` origins.

Details in [HTTPS on the LAN](../self-hosting/lan-https.md) and [Cloudflare Tunnel and Tailscale](../self-hosting/tunnels.md).

## Trace AI

### Trace returns "model not found" or a Gemini 404

**Symptom:** Trace works with some providers but Gemini calls fail with 404, or the UI shows a "model not found" error after an upgrade.

**Cause:** Google retires Gemini model IDs regularly. The saved user setting still points at a retired one (`gemini-1.5-flash`, `gemini-1.5-pro`, `gemini-2.0-flash`, `gemini-2.0-flash-lite`). The server keeps a `GEMINI_RETIRED` set and auto-remaps to the current default (`gemini-2.5-flash`) on next call, but the remap runs once at boot; a saved user preference can still slip through.

**Fix:** Restart the container to trigger the remap, then open **Settings > Trace AI** and either pick a fresh model from the dropdown or clear the saved setting. If you want to override with a model that is not on the built-in list, use the `__custom__` option (`AI_MODEL_CUSTOM` on CookTrace) and type the ID by hand. See [Model list and Gemini retirement remap](../trace/models.md).

## Backups

### Full-backup restore fails behind a reverse proxy

??? details "Symptom, cause, and the two proxy caps to check"
    **Symptom:** The full-backup ZIP downloads fine but the restore upload fails part-way through, often with a 413 or a truncated request.

    **Cause:** The reverse proxy in front of your instance is capping the request body below the backup size.

    **Fix:** Two caps to check:

    - **Cloudflare free tier** proxies bodies at up to 100 MB. Restore uploads larger than that will not fit. Bypass Cloudflare for the restore (Tailscale, a direct LAN address, or temporarily "grey-cloud" the DNS record), or pay for a plan with a higher cap.
    - **nginx** defaults `client_max_body_size` to 1 MB. Bump it explicitly in your server block: `client_max_body_size 2048m;` or whatever comfortably exceeds your backup size. Caddy and Traefik have generous defaults but check your specific configuration.

    See [Backups and restore](../self-hosting/backups.md) and [Reverse-proxy recipes](../self-hosting/proxy-recipes.md) for full snippets.

## OIDC / SSO

### OIDC login works but the user has no admin rights

**Symptom:** Users authenticate through your IdP fine, get logged in, but land with the default `user` role even though they are in the group that is supposed to grant admin.

**Cause:** The group claim your IdP emits is not matching what the app is looking for. Two things have to line up: the claim name (`OIDC_PROVIDER_1_ADMIN_GROUP_CLAIM`, default `groups`) and the value inside that claim (`OIDC_PROVIDER_1_ADMIN_GROUP_VALUE`).

**Fix:** Decode the ID token your IdP issues (the OIDC provider's admin console usually has a debug view, or use jwt.io) and confirm the exact claim name and value. Update the two env vars to match, restart the container, and have the user log out and back in. Role elevation runs on each sign-in, so an existing session will not pick up the new mapping until the next login. Full walkthroughs per IdP live under [OIDC recipes](../auth/oidc.md).

### Rotating JWT_SECRET locks out users mid-OIDC-flow

**Symptom:** After rotating `JWT_SECRET` the login page rejects fresh OIDC callbacks with a "state mismatch" or "invalid token" error, even though the IdP end of the handshake completed.

**Cause:** The app signs OIDC state and nonce tokens with `JWT_SECRET`. Rotating the secret invalidates every in-flight state token, so any user mid-redirect from the IdP gets rejected on the callback. Existing sessions are also invalidated, which is the whole point of the rotation.

**Fix:** Warn users before rotating, or accept the brief outage. Once the container is back up on the new secret, users just need to re-login from scratch (a fresh state token is issued on the new redirect). If you also want the encrypted OIDC client secrets in the database to survive rotation, set `TOKEN_ENC_KEY` explicitly to something stable; otherwise `TOKEN_ENC_KEY` derives from `JWT_SECRET` and rotates with it, requiring you to re-enter every OIDC provider secret in the admin UI. See [Session lifetime and password policy](../auth/sessions.md).

## NutriTrace-specific

### OFF local mirror errors with `EISDIR`

**Symptom:** On NutriTrace, the OFF (Open Food Facts) mirror refresh fails with an `EISDIR: illegal operation on a directory` error, and food lookups against the local mirror return empty.

**Cause:** `OFF_LOCAL_DB` must point at a **file path**, not a directory. If the compose file bind-mounts a host directory onto the path in `OFF_LOCAL_DB`, Docker auto-creates a directory at that path on first start, and every subsequent file open fails.

**Fix:** In your compose file, mount the *containing* directory (something like `./off-mirror:/data/off-mirror`) and set `OFF_LOCAL_DB=/data/off-mirror/off.duckdb` to point at the file inside it. Do not set `OFF_LOCAL_DB` to the mount point itself. See [Open Food Facts](../nutritrace/off.md).

### Diary items disappear after force-killing the app overnight

**Symptom:** You log a few meals on the NutriTrace Android app in the evening, force-kill the app or the OS kills it in the background, and by morning the meals are gone. Water entries logged the same evening are still there.

**Cause:** On older NT Android builds, diary item writes were debounced to SQLite while water writes flushed immediately. If the app process died before the debounce timer fired, the diary rows never made it out of the JavaScript layer into SQLite storage. Water was unaffected because it took a different code path.

**Fix:** Upgrade to NutriTrace `1.0.3` (Android `versionCode 128`) or newer. Diary writes now flush on `pause`, `visibilitychange`, and `pagehide` events, matching the water write path. If you are already on a current build, confirm the version in **Settings > About** and re-check that the affected day is still absent from the server after a manual sync (from your PWA at, for example, `https://nutritrace.example.com`); a genuinely-missing diary should also be missing there.

## When nothing else helps

- Reproduce with `LOG_LEVEL=debug` in the server container and grab the last 200 lines.
- On the phone, turn on **Settings > Diagnostics > Verbose** and copy the diagnostic log.
- Open an issue on the relevant repo ([`TraceApps/cooktrace`](https://github.com/TraceApps/cooktrace/issues), [`TraceApps/lifttrace`](https://github.com/TraceApps/lifttrace/issues), [`TraceApps/nutritrace`](https://github.com/TraceApps/nutritrace/issues)) with the version, the log snippet, and the exact steps to reproduce. Redact anything sensitive; wellness logs in particular contain PHI (HRV, resting heart rate, sleep, calories).

## Related

- [How sync works](../mobile/sync.md)
- [Plain-HTTP LAN (INSECURE_COOKIES)](../getting-started/lan-http.md)
- [HTTPS on the LAN](../self-hosting/lan-https.md)
- [OIDC / SSO overview](../auth/oidc.md)
- [Model list and Gemini retirement remap](../trace/models.md)
