# Env vars (NoteTrace-specific)

NoteTrace reads the same shared variables as the other Trace apps (`PORT`, `BASE_URL`, `JWT_SECRET`, `DB_PATH`, `UPLOADS_PATH`, `SMTP_*`, `OIDC_*`, `AI_*`, `INSECURE_COOKIES`, `LOG_LEVEL`, `RECOVERY_TOKEN`, `TOKEN_ENC_KEY`, `<NAME>_FILE` secrets, and the rest). Those live on the shared [environment reference](../self-hosting/env-vars.md). This page lists the values that are specific to NoteTrace or worth calling out for it.

## Container

| Env var | Default | Purpose |
| --- | --- | --- |
| `PORT` | `3001` | Port the server listens on inside the container. The sample compose file maps it to host port `3004`. |
| `DB_PATH` | `/data/db/notetrace.db` | SQLite database file. Mount `/data/db` as a volume. |
| `UPLOADS_PATH` | `/data/uploads` | Uploaded images and server-side backups. Mount `/data/uploads` as a volume. |
| `BASE_URL` | (empty) | Subpath mount, for example `/notetrace`. |

## Integrations

| Env var | Default | Purpose |
| --- | --- | --- |
| `WEBHOOKS_ENABLED` | (unset) | Set to `1` to turn on outgoing [webhooks](webhooks.md) (`note.created`, `checklist.completed`, `reminder.fired`). |
| `ALLOW_PRIVATE_WEBHOOK_URLS` | (unset) | Set to `1` to allow webhook targets on private or loopback addresses, such as a Home Assistant container on the same Docker network. |
| `MCP_ENABLED` / `MCP_WRITE_ENABLED` / `MCP_DESTROY_ENABLED` | (unset) | Model Context Protocol endpoint and its write and destructive tiers. See [MCP](mcp.md). |
| `ALLOW_PRIVATE_COOKTRACE_URLS` | (unset) | Set to `1` so [Send to CookTrace](cooktrace.md) can reach a CookTrace on a LAN, loopback, or Docker network address. |

## Trace

| Env var | Default | Purpose |
| --- | --- | --- |
| `AI_TRANSCRIBE_MODEL` | provider default | Speech-to-text model for [voice notes](trace.md#voice) when Trace is set by env vars: `gpt-4o-mini-transcribe` on OpenAI, `whisper-1` on an OpenAI-compatible server. Gemini uses `AI_MODEL`. |

## Backup

| Env var | Default | Purpose |
| --- | --- | --- |
| `BACKUP_UPLOAD_MAX_MB` | `512` | Upload size cap (MB) for restoring a full-backup zip. |
| `BACKUP_SCHEDULE` | (unset) | `off`, `daily`, `weekly`, or `monthly`. Locks the Settings field when set. |
| `BACKUP_TIME` | (unset) | Auto-backup time (`HH:MM`, container time zone). Locks the Settings field. |
| `BACKUP_RETENTION` | (unset) | How many auto-backups to keep. |

## Path convenience (Docker Compose only)

| Env var | Default | Purpose |
| --- | --- | --- |
| `DATA_DB_PATH` | (compose) | Host folder for the database volume. Read by the compose file, not by the app. |
| `DATA_UPLOADS_PATH` | (compose) | Host folder for the uploads volume. |

## No env vars needed

Reminders, sharing, and imports have no server settings. Server reminder delivery runs whenever a user has a push service or a `reminder.fired` webhook set up. Imports are sent in batches of up to 500 notes, so they fit within the server's normal request limits without tuning.

## Related

- [Install with Docker Compose](../getting-started/compose.md)
- [Docker Secrets (*_FILE)](../self-hosting/secrets.md)
