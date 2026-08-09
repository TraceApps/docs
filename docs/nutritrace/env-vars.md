# NutriTrace env vars (app-specific)

This page lists **NutriTrace-specific** environment variables (the ones that only apply to NutriTrace, or that behave differently here than in the other Trace apps). For the shared cross-app env vars (`DB_PATH`, `PORT`, `BASE_URL`, `JWT_SECRET`, `INSECURE_COOKIES`, `LOG_LEVEL`, `MAX_SESSION_HOURS`, SMTP, OIDC, Docker Secrets pattern), see [Environment reference](../self-hosting/env-vars.md).

Anything on this page can also be supplied via the `<NAME>_FILE=/run/secrets/...` pattern (`server/docker-entrypoint.sh` reads the file and exports the value before Node starts).

## Federation API

| Var | Default | Purpose |
|---|---|---|
| `API_RATE_LIMIT_PER_MIN` | `60` | Requests per minute per federation-API bearer token. Middleware in `server/middleware/bearer-auth.js`. Applies to MCP tokens too. |

## Model Context Protocol (MCP)

Three independent flags, each guarding a bigger surface. Off by default; the endpoint returns 404 until `MCP_ENABLED=1`. Every flag change needs a container restart. See the [MCP setup guide](mcp.md) for the full setup path.

| Var | Default | Purpose |
|---|---|---|
| `MCP_ENABLED` | `0` | Turns on `/api/mcp` and registers the 5 read tools. Tokens still need the `mcp:read` scope. |
| `MCP_WRITE_ENABLED` | `0` | Registers the 4 additive log tools (`log_food`, `log_water`, `log_meal`, `log_body_stat`). Tokens need `mcp:write`. |
| `MCP_DESTROY_ENABLED` | `0` | Registers the 3 destructive tools (`delete_diary_entry`, `edit_diary_entry`, `create_food`). Tokens need `mcp:destroy`, AND every call must include `confirm: true`. |
| `ALLOWED_ORIGINS` | — | Comma-separated list of browser Origins allowed to call `/api/mcp`. Server-to-server clients (Claude Desktop's HTTP bridge etc.) send no Origin header and always pass, so most self-hosters leave this empty. `*` is refused by design (DNS-rebinding defense). |

## AI Assistant (env-lock)

Setting any of these locks the corresponding UI field for all users. UI shows a lock badge. Read in `server/ai.js` / `server/routes/ai.js`.

| Var | Purpose |
|---|---|
| `AI_ENABLED` | `true` auto-enables Trace for every user. Users can still turn it off per-user unless combined with the other vars below. |
| `AI_PROVIDER` | `claude` / `openai` / `gemini` / `oai-compat`. |
| `AI_API_KEY` | Provider API key. Required when `AI_PROVIDER` is set. |
| `AI_MODEL` | Model id. **Required** when `AI_PROVIDER=oai-compat`; optional otherwise (defaults to the provider's current recommended model). |
| `AI_BASE_URL` | Endpoint URL. **Required** when `AI_PROVIDER=oai-compat` (Ollama, LM Studio, LocalAI, vLLM, llama.cpp, DeepSeek, Groq, Together, Mistral, OpenRouter, and so on). |

When any AI env var is set, the server proxies AI calls; the API key never touches the browser. Without env-lock, the browser calls the provider directly with the user's own key.

## Open Food Facts local mirror

Read in `server/lib/off-local.js`.

| Var | Purpose |
|---|---|
| `OFF_LOCAL_DB` | In-container path to the Parquet or DuckDB file (e.g. `/data/off-mirror/off.parquet`). Presence enables the mirror. |
| `OFF_LOCAL_ONLY` | `1` = air-gap; refuse the public OFF API entirely. |
| `OFF_LOCAL_URL` | Override the download source. Defaults to the Hugging Face `food.parquet`. |
| `OFF_LOCAL_DB_HOST_PATH` | Compose-only var used as the host side of the bind-mount default (`${OFF_LOCAL_DB_HOST_PATH:-./off-mirror}:/data/off-mirror`). |

See [Open Food Facts](off.md) for the full walkthrough (~7-8 GB download, refresh schedule, ODbL banner).

## Wellness (Fitbit / Google Health / Withings / Garmin)

**None.** All wellness integrations are configured **per user**, not via env vars. Each user pastes their own Client ID / Secret pair for the wearable they use, because Fitbit / Google Cloud / Withings / Garmin all issue credentials per developer account rather than per app instance. Callbacks are handled at `/api/wellness/<provider>/callback` and secrets are AES-GCM encrypted at rest with `TOKEN_ENC_KEY`.

See [Wellness and wearables](wellness.md), [Fitbit → Google Health migration](google-health.md), [Withings](withings.md), and [Health Connect](health-connect.md).

## USDA and Mealie

**None.** Both are per-user under **Settings → Connected Services**. USDA needs a free `api.data.gov` key; Mealie needs the base URL of your running Mealie plus a personal API token. See [USDA and Mealie integration](usda-mealie.md).

## Full Backup schedule

Setting any of these lands the container in "env-configured backup" mode; the admin UI shows the schedule as read-only. Read in `server/routes/full-backup.js`.

| Var | Purpose |
|---|---|
| `BACKUPS_PATH` | Directory for full-backup ZIPs. Defaults to `${UPLOADS_PATH}/backups` (inside the uploads volume). |
| `BACKUP_SCHEDULE` | Cadence: `daily` / `weekly` / `monthly` / cron expression. |
| `BACKUP_TIME` | Local time-of-day for the schedule (`HH:MM`). |
| `BACKUP_RETENTION` | How many recent backups to keep before rotating out. |
| `BACKUP_UPLOAD_MAX_MB` | Max size of an uploaded backup ZIP for restore-from-upload. Default `512`. |

## What is NOT NutriTrace-specific

For reference, these are the shared cross-Trace-app env vars covered on the [Environment reference](../self-hosting/env-vars.md) page and not repeated here: `DB_PATH`, `UPLOADS_PATH`, `PORT`, `BASE_URL`, `LOG_LEVEL`, `NODE_ENV`, `JWT_SECRET`, `TOKEN_ENC_KEY`, `RECOVERY_TOKEN`, `INSECURE_COOKIES`, `MAX_SESSION_HOURS`, `SMTP_*`, `OIDC_*`.

## Related

- [Environment reference (shared)](../self-hosting/env-vars.md)
- [Settings reference](settings.md)
- [Open Food Facts](off.md)
- [Wellness and wearables](wellness.md)
