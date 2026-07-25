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
| `BACKUPS_PATH` | `<UPLOADS_PATH>/backups` | Where full-backup zips write. Change only if you want backups on a different volume. |

## Path convenience (Docker compose only)

| Env var | Default | Purpose |
| --- | --- | --- |
| `DATA_DB_PATH` | (compose) | Host-side bind mount for the DB directory. Not read by the app; the compose file passes it through. |
| `DATA_UPLOADS_PATH` | (compose) | Host-side bind mount for uploads. Same story. |

## Push notification origin

| Env var | Default | Purpose |
| --- | --- | --- |
| `APP_ORIGIN` / `PUBLIC_URL` | (unset) | Used by `server/lib/push-notify.js` to build absolute links inside push messages (so tapping a notification lands on the right host). Set to your public URL if you use push. |

## Not env vars but adjacent

- **`google-services.json`** at `android/app/` (optional): enables Firebase push in the Android build. Only relevant if you rebuild the APK yourself.

## Related

- [Shared self-hosting env reference](../self-hosting/env-vars.md)
- [Settings reference](settings.md)
- [Import](import.md)
- [Backups](../self-hosting/backups.md)
