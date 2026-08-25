# Environment reference

Every knob the three apps expose is in the table below. Defaults come from the actual server code, not the sample compose files, so if a README example and this page disagree, this page wins. The last three columns show which app reads which variable: `Y` means the app respects it, blank means it does not.

## Env-lock: what setting a var does to the UI

Almost every runtime setting has two homes: an environment variable, and a field inside the Settings UI backed by the `app_config` table. When a variable is set at container start, the server writes the value into `app_config` **and** marks the field as env-locked. Two things follow.

First, the client marks the field read-only (with a small lock badge) and refuses to save changes to it. Any admin who tries to edit an env-locked value gets a "locked by environment variable" error from `GET /api/app-config/env-locks`. Second, if you later unset the variable and restart the container, the lock clears and the field becomes editable again from the UI.

The pattern applies to SMTP, AI, OIDC providers, backup schedule, and (NutriTrace only) Open Food Facts (OFF) mirror settings. Two implications worth internalising: (1) once you decide to manage a setting from env, keep managing it from env, because someone else's UI save will be silently overridden on the next boot; (2) secrets live only on the server. The browser never receives the raw `AI_API_KEY`, `SMTP_PASS`, or `OIDC_CLIENT_SECRET`; the API returns eight-dot placeholders instead.

Any variable in the tables below can also be provided via Docker secrets by suffixing `_FILE`. `SMTP_PASS_FILE=/run/secrets/smtp_pass`, `AI_API_KEY_FILE=/run/secrets/openai_key`, `JWT_SECRET_FILE=/run/secrets/jwt`, and so on. Setting both `NAME` and `NAME_FILE` is a boot-time error. See [Docker secrets](secrets.md) for the full pattern.

## Core

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `PORT` | `3001` (CT/NT), `3003` (LT) | Container listen port. Change if you remap. | Y | Y | Y |
| `BASE_URL` | (empty) | Subpath mount, e.g. `/cooktrace`. Must start with `/`, no trailing slash. Emitted to the browser as `window.__<APP>_CONFIG__.basePath`. | Y | Y | Y |
| `LOG_LEVEL` | `info` | `error`, `warn`, `info`, `debug`. | Y | Y | Y |
| `NODE_ENV` | (empty) | Set to `production` in real deployments. Refuses to boot without `JWT_SECRET`. | Y | Y | Y |

## Storage

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `DB_PATH` | `./<app>.db` | SQLite file. In the container, mount to `/data/db/<app>.db`. WAL mode is enabled at boot. | Y | Y | Y |
| `UPLOADS_PATH` | `./uploads` | Upload directory. Mount to `/data/uploads`. Served publicly at `/uploads/*` (Android WebView can't send `Authorization` on `<img src>`). | Y | Y | Y |
| `BACKUPS_PATH` | `<UPLOADS_PATH>/backups` | Where the built-in "Full Backup" writes ZIPs. | Y | Y | Y |

## Auth and crypto

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `JWT_SECRET` | insecure dev fallback | Signs the auth cookie and Bearer tokens. Required when `NODE_ENV=production`. Long random string; never share across apps. | Y | Y | Y |
| `TOKEN_ENC_KEY` | HKDF-derived from `JWT_SECRET` | AES-GCM key at rest for OIDC client secrets and (NT) wearable OAuth tokens. Set explicitly if you plan to rotate `JWT_SECRET`. | Y | Y | Y |
| `RECOVERY_TOKEN` | unset | Enables `POST /api/auth/recover` from the login page. Passphrase compare is constant-time. Without this the endpoint is disabled. | Y | Y | Y |
| `INSECURE_COOKIES` | unset | Set `1` on plain-HTTP LAN deployments so the `Secure` attribute comes off the cookie. Otherwise every request after login 401s. | Y | Y | Y |
| `MAX_SESSION_HOURS` | `8760` (1 year) | Cap on JWT and cookie lifetime. Per-user overrides may be lower. | Y | Y | Y |

## SMTP

The whole group is optional. If `SMTP_HOST` is set, the entire Email tile locks in the admin UI and the DB values are ignored. `SMTP_PASS` is never returned to the browser; the API sends eight dots instead.

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `SMTP_HOST` | unset | Hostname of your relay. Setting this triggers env-lock for the whole group. | Y | Y | Y |
| `SMTP_PORT` | `587` | 587 for STARTTLS, 465 for implicit TLS. | Y | Y | Y |
| `SMTP_SECURE` | `false` | `true` for SSL on 465, `false` (default) for STARTTLS on 587. | Y | Y | Y |
| `SMTP_USER` | unset | Login for the relay. | Y | Y | Y |
| `SMTP_PASS` | unset | Password for the relay. Use `SMTP_PASS_FILE` in production. | Y | Y | Y |
| `SMTP_FROM` | unset | RFC 5322 From header, e.g. `"CookTrace" <noreply@example.com>`. | Y | Y | Y |

## Trace AI

Same envelope across all three apps: pick a provider, hand it a key, optionally pin a model. `AI_ENABLED=true` auto-enables Trace for every user; leave it unset if you want each user to opt in themselves.

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `AI_ENABLED` | unset | `true` = auto-enable Trace for all users. | Y | Y | Y |
| `AI_PROVIDER` | unset | `claude`, `openai`, `gemini`, `oai-compat`. `oai-compat` covers Ollama, LM Studio, vLLM, DeepSeek, Groq, LocalAI. | Y | Y | Y |
| `AI_API_KEY` | unset | Required for cloud providers. Optional for `oai-compat` (Ollama runs keyless). | Y | Y | Y |
| `AI_MODEL` | provider default | Optional for cloud, required for `oai-compat`. | Y | Y | Y |
| `AI_BASE_URL` | unset | Required for `oai-compat`. Reachable from inside the container (Docker network hostnames work). | Y | Y | Y |

## OIDC

The shorthand `OIDC_*` variables alias `OIDC_PROVIDER_1_*`. For multi-provider setups, add `OIDC_PROVIDER_2_*`, `OIDC_PROVIDER_3_*`, and so on. Any provider defined by env shows a lock badge in the admin UI and refuses edits. See [OIDC / SSO](../auth/oidc.md) for full worked examples with Authentik, Pocket-ID, Keycloak, and Google Workspace.

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `OIDC_ISSUER` | unset | Discovery URL root. | Y | Y | Y |
| `OIDC_CLIENT_ID` | unset | Client ID from your IdP. | Y | Y | Y |
| `OIDC_CLIENT_SECRET` | unset | Client secret. Encrypted at rest with `TOKEN_ENC_KEY`. | Y | Y | Y |
| `OIDC_DISPLAY_NAME` | unset | Button label on the login page. | Y | Y | Y |
| `OIDC_LOGO_URL` | unset | Optional icon on the login button. | Y | Y | Y |
| `OIDC_SCOPE` | `openid profile email` | Space-separated scopes. | Y | Y | Y |
| `OIDC_REDIRECT_URIS` | unset | Comma-separated. Include both the PWA callback and the `<app>://oidc-callback` deep link if you use Android. | Y | Y | Y |
| `OIDC_TOKEN_AUTH_METHOD` | `client_secret_post` | Also `client_secret_basic` or `none`. | Y | Y | Y |
| `OIDC_ADMIN_GROUP_CLAIM` | unset | Claim name that lists groups, e.g. `groups`. | Y | Y | Y |
| `OIDC_ADMIN_GROUP_VALUE` | unset | Membership in this group promotes the user to admin on every sign-in. | Y | Y | Y |
| `OIDC_AUTO_LINK` | `1` | Link SSO identity to an existing account with the same verified email. | Y | Y | Y |
| `OIDC_AUTO_REGISTER` | `0` | Create a new account on first SSO sign-in. | Y | Y | Y |
| `OIDC_IS_ACTIVE` | `1` | Set `0` to disable the provider without removing the config. | Y | Y | Y |
| `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN` | `1` | Set `0` for SSO-only mode (password login disabled server-wide). Locks the admin toggle. | Y | Y | Y |

## Backup

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `BACKUP_SCHEDULE` | `off` | `off`, `daily`, `weekly`, `monthly`. Setting any of the three backup vars env-locks the tile. | Y | Y | Y |
| `BACKUP_TIME` | `03:00` | `HH:MM`, container timezone. | Y | Y | Y |
| `BACKUP_RETENTION` | `7` | Number of backups to keep. Clamped to 1..99. | Y | Y | Y |
| `BACKUP_UPLOAD_MAX_MB` | `512` | Cap on restore-upload ZIP size (multer). | Y | Y | Y |

## Networking

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `HTTP_PROXY` | unset | Forward proxy for outbound HTTP requests from the server. Standard curl / requests convention: `http://proxy.host:3128`. Lowercase `http_proxy` also accepted. |  Y  |  Y  |  Y  |
| `HTTPS_PROXY` | unset | Forward proxy for outbound HTTPS requests. Typically the same value as `HTTP_PROXY`. Lowercase `https_proxy` also accepted. |  Y  |  Y  |  Y  |
| `NO_PROXY` | unset | Comma-separated list of hosts / domain suffixes / IPs that bypass the proxy. Example: `localhost,127.0.0.1,.internal,off-mirror.local`. Lowercase `no_proxy` also accepted. |  Y  |  Y  |  Y  |

Setting any of the three at container start installs an undici `EnvHttpProxyAgent` as the global fetch dispatcher and monkey-patches `globalThis.fetch` to route through it, so every outbound request from the server obeys the proxy without per-call-site changes. When none are set, fetch behaves as before with no dep cost. Startup logs one line confirming which of the three are active.

CT-specific note: the Enhanced URL Import mode spawns a Python `recipe-scrapers` subprocess for site-specific extractors. The subprocess inherits env vars, so the proxy settings are visible to it, but whether outbound calls from Python honor them depends on which HTTP client `recipe-scrapers` uses internally.

## App-specific

| Variable | Default | Purpose | CT | LT | NT |
|---|---|---|---|---|---|
| `IMPORT_ZIP_MAX_MB` | `512` | Cap on the bulk recipe-import ZIP. |  Y  |     |     |
| `PYTHON_BIN` | `python3` | Interpreter used by the recipe-scrapers bridge. |  Y  |     |     |
| `EXERCISE_SOURCES` | (empty) | Comma-separated auto-seed IDs: `wger`, `free-db`, `exercisedb`, `exercisedb-oss`. Empty triggers the wizard picker. |     |  Y  |     |
| `EXERCISEDB_OSS_URL` | `https://oss.exercisedb.dev` | Override the ExerciseDB-OSS mirror. |     |  Y  |     |
| `ALLOW_PRIVATE_RADIO_URLS` | `0` | Set `1` to allow the radio proxy to reach RFC 1918 / loopback / IPv6 ULA (LAN Icecast, Navidrome, Subsonic). |     |  Y  |     |
| `OFF_LOCAL_DB` | unset | In-container path to the OFF Parquet or DuckDB mirror (e.g. `/data/off-mirror/off.parquet`). |     |     |  Y  |
| `OFF_LOCAL_ONLY` | unset | Set `1` for air-gap mode: refuse the public OFF fallback. |     |     |  Y  |
| `OFF_LOCAL_URL` | HuggingFace `food.parquet` | Override the download source used to refresh the mirror. |     |     |  Y  |
| `API_RATE_LIMIT_PER_MIN` | `60` | Federation API Bearer-token rate cap, per token. |     |     |  Y  |

## Related

- [Docker secrets](secrets.md)
- [Backups and restore](backups.md)
- [Reverse proxy](../getting-started/reverse-proxy.md)
- [OIDC / SSO](../auth/oidc.md)
