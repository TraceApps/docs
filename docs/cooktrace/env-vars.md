# Env vars (CookTrace-specific)

Only CookTrace-specific env vars are listed here. Shared env vars (`PORT`, `BASE_URL`, `JWT_SECRET`, `DB_PATH`, `UPLOADS_PATH`, `SMTP_*`, `OIDC_*`, `AI_*`, `INSECURE_COOKIES`, `LOG_LEVEL`, `RECOVERY_TOKEN`, `TOKEN_ENC_KEY`, `<NAME>_FILE` secrets, etc.) live on the shared [self-hosting env reference](../self-hosting/env-vars.md).

## Import

| Env var | Default | Purpose |
| --- | --- | --- |
| `IMPORT_ZIP_MAX_MB` | `512` | Upload size cap (MB) for bulk Mealie / Tandoor / Paprika import zips. Applied at `POST /api/recipes/import-zip/scan` and the bulk-scan endpoint. Bump if you routinely import bigger libraries. |
| `PYTHON_BIN` | `python3` | Python binary shelled by `server/lib/recipe-scrapers-bridge.js` for the Enhanced URL import tier. Only touch this if the baked-in Python isn't on `PATH` (custom base image, etc.). |

## Backup

| Env var | Default | Purpose |
| --- | --- | --- |
| `BACKUP_UPLOAD_MAX_MB` | `512` | Upload size cap (MB) for **Upload & Restore** of a full-backup zip. Not the same as `IMPORT_ZIP_MAX_MB`. |
| `BACKUP_SCHEDULE` | (unset) | Auto-backup cadence: `off` \| `daily` \| `weekly` \| `monthly`. When set, locks the Settings UI field. |
| `BACKUP_TIME` | `02:00` | Auto-backup time of day (`HH:MM`, container timezone). Locks the UI when set. |
| `BACKUP_RETENTION` | (unset) | How many auto-backups to keep (older auto-purged). Locks the UI when set. |
| `BACKUPS_PATH` | `<UPLOADS_PATH>/backups` | Where full-backup zips write. Change only if you want backups on a different volume. Wherever it points inside `UPLOADS_PATH`, that directory is never served publicly. |

## Integrations

All off by default.

| Env var | Default | Purpose |
| --- | --- | --- |
| `MCP_ENABLED` / `MCP_WRITE_ENABLED` / `MCP_DESTROY_ENABLED` | `0` | The three MCP tiers. See [Model Context Protocol](mcp.md), which also covers `ALLOWED_ORIGINS`. |
| `PUBLIC_API_ENABLED` | `0` | Turns on the read routes of the [public REST API](public-api.md). |
| `PUBLIC_API_WRITE_ENABLED` | `0` | Also turns on its write routes. Needs `PUBLIC_API_ENABLED`. |
| `WEBHOOKS_ENABLED` | `0` | Turns on outgoing [webhooks](webhooks.md). |
| `ALLOW_PRIVATE_WEBHOOK_URLS` | `0` | Lets a webhook target a private or loopback address (a Home Assistant container on the same Docker network, for example). Link-local and cloud-metadata addresses stay blocked. |
| `ALLOW_PRIVATE_RECIPE_URLS` | `0` | Lets every account import a recipe from a link on your own network (a LAN Mealie, say). An admin may; other accounts, and a single-user install (which has no sign-in), only with this set. |
| `ALLOW_PRIVATE_AI_URLS` | `0` | Lets smart import use an AI address on your own network that an account set itself (an Ollama on the LAN). An admin may; other accounts, and a single-user install (which has no sign-in), only with this set. The server's own AI (`AI_*` variables) never needs it. |

## Path convenience (Docker compose only)

| Env var | Default | Purpose |
| --- | --- | --- |
| `DATA_DB_PATH` | (compose) | Host-side bind mount for the DB directory. Not read by the app; the compose file passes it through. |
| `DATA_UPLOADS_PATH` | (compose) | Host-side bind mount for uploads. Same story. |

## Push notification origin

| Env var | Default | Purpose |
| --- | --- | --- |
| `APP_ORIGIN` / `PUBLIC_URL` | (unset) | The address in links: push messages, and (`PUBLIC_URL`, from 1.4.1) password reset, invite and sharing emails. Set to the full address people open CookTrace at. |

## Not env vars but adjacent

- **`google-services.json`** at `android/app/` (optional): enables Firebase push in the Android build. Only relevant if you rebuild the APK yourself.

## Related

- [Shared self-hosting env reference](../self-hosting/env-vars.md)
- [Settings reference](settings.md)
- [Import](import.md)
- [Backups](../self-hosting/backups.md)
