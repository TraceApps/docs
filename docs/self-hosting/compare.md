# How the three apps compare

CookTrace, LiftTrace, and NutriTrace are siblings by design: same Node + Express 5 + better-sqlite3 + Svelte stack, same Docker layout, same reverse-proxy story, same auth and Trace-AI plumbing. The differences that matter to a self-hoster are small but real. This page collects them in one table so you can pick the compose file, port, and base image for each install without hunting through three READMEs.

## At a glance

| Item | CookTrace | LiftTrace | NutriTrace |
|---|---|---|---|
| Docker image | `ghcr.io/traceapps/cooktrace` | `ghcr.io/traceapps/lifttrace` | `ghcr.io/traceapps/nutritrace` |
| Base image | `node:20-alpine` | `node:20-alpine` | `node:20-slim` (Debian) |
| Container port | `3001` | `3003` | `3001` |
| Sample host port | `3000` | `3002` | `3000` |
| DB mount | `/data/db` | `/data/db` | `/data/db` |
| Uploads mount | `/data/uploads` | `/data/uploads` | `/data/uploads` |
| Extra mount | none | none | `/data/off-mirror` (optional) |
| Auth cookie | `ct_token` | `lt_token` | `nutritrace_token` |
| Subpath env | `BASE_URL=/cooktrace` | `BASE_URL=/lifttrace` | `BASE_URL=/nutritrace` |
| Health endpoint | `GET /api/health` | `GET /api/health` | `GET /api/health` |
| Python bundled | yes (recipe-scrapers) | yes (better-sqlite3 build) | yes (better-sqlite3 build) |
| Android reminders | JS `LocalNotifications` | JS `LocalNotifications` | native WorkManager |
| SMTP settings live in | Settings, Email | Settings, Email | Settings, Authentication |
| Federation | pulls foods from NT | none | serves foods to CT |
| App-only env vars | `IMPORT_ZIP_MAX_MB` | `EXERCISE_SOURCES`, `EXERCISEDB_OSS_URL`, `ALLOW_PRIVATE_RADIO_URLS` | `OFF_LOCAL_DB`, `OFF_LOCAL_ONLY`, `OFF_LOCAL_URL`, `API_RATE_LIMIT_PER_MIN` |

All three publish multi-arch images (`linux/amd64` + `linux/arm64`) with the same tag matrix: `X.Y.Z`, `X.Y`, `X`, `latest`, `dev`. Legacy `X.Y.Z-rc.N` tags stay pinned indefinitely.

## Why NutriTrace uses a Debian base

Alpine's musl libc cannot load the DuckDB native bindings, and NutriTrace uses DuckDB to query the optional Open Food Facts local mirror (`OFF_LOCAL_DB`, roughly 7 to 8 GB Parquet). To keep a single image regardless of whether the mirror is on, the container is built on `node:20-slim` instead of `node:20-alpine`. Expect a slightly larger image in return for the OFF air-gap option. CookTrace and LiftTrace have no glibc-only dependency and stay on Alpine.

## Why CookTrace bundles a Python interpreter

Recipe import from third-party sites goes through [recipe-scrapers](https://github.com/hhursev/recipe-scrapers), a Python package with 300+ site-specific extractors. CookTrace's Dockerfile installs `python3` plus `recipe-scrapers`, and the Node process shells out via `PYTHON_BIN` (default `python3`) when a scraper matches. LiftTrace and NutriTrace install `python3` only as a build-time dependency for `better-sqlite3` and never invoke it at runtime.

## Android reminder backend

NutriTrace's Android build uses a native WorkManager worker for meal, water, weigh-in, and wind-down reminders (gated by the internal `_USE_NATIVE_WORKER` toggle). This survives WebView suspension and battery-optimisation kills better than the Capacitor `LocalNotifications` scheduler. CookTrace and LiftTrace still route reminders through the JS `LocalNotifications` plugin. If you run all three and reminders occasionally miss on the two lighter apps, this is why.

## LiftTrace's radio LAN allowance

Only LiftTrace has a built-in radio player. If you run a Subsonic, Navidrome, or Icecast server on the same LAN as LiftTrace, set `ALLOW_PRIVATE_RADIO_URLS=1` so the radio proxy stops refusing RFC 1918 and IPv6 ULA addresses. CookTrace and NutriTrace have no equivalent knob because they have no radio.

## SMTP form location

CookTrace and LiftTrace put the SMTP form under Settings, Email. NutriTrace keeps it inside Settings, Authentication (same fields, same test-email button, just a different tile). Every env-lock and every save behaves the same way; the tile just lives one section over.

## Related

- [Environment reference](env-vars.md)
- [Reverse proxy](../getting-started/reverse-proxy.md)
- [Backups and restore](backups.md)
