# Scheduled auto-backups

The three apps have a built-in backup scheduler that runs inside the container: no host cron, no external timer, no sidecar. Turn it on with three env vars or from the admin UI, and the container writes a fresh ZIP on the interval you pick.

For the anatomy of the ZIP itself and the manual button, see [Backups and restore](backups.md). This page covers the scheduling knobs and the env-lock behaviour.

## The three env vars

| Variable | Values | Default | Notes |
|---|---|---|---|
| `BACKUP_SCHEDULE` | `off`, `daily`, `weekly`, `monthly` | `off` | Off means the scheduler tick does nothing; the manual button still works. |
| `BACKUP_TIME` | `HH:MM` (24-hour) | `03:00` | Runs at this time in the container's timezone. Set `TZ=Europe/Paris` if the host default is not what you want. |
| `BACKUP_RETENTION` | integer `1..99` | `7` | After a successful run, ZIPs beyond the newest N are pruned. Clamped: `0` becomes `1`, `100` becomes `99`. |

Weekly runs on the day the schedule was first activated; monthly runs on the same calendar day each month. If the container is stopped when a run is due, the scheduler does not backfill; the next tick after start-up will pick up the next scheduled window.

## Where the ZIPs land

`/data/backups/` inside the container, which is `<UPLOADS_PATH>/backups` by default (override with `BACKUPS_PATH`). Because the default sits under the uploads volume, the archives survive container restarts as long as you have mounted `/data/uploads` to a host volume, which the sample compose does.

Filenames look like `<app>-backup-2026-07-25T03-00-00.zip`. The retention prune sorts by filename (timestamp is in the name), so a stray `cp -p` will not reshuffle the keep-order.

## In-app settings location

Admin only, under **Settings, Backup**. The tile shows the current schedule, the time-of-day input, and retention count, plus a "Last automatic backup" line and a red banner if the last run failed. When any of the three env vars is set, the fields turn read-only and pick up a lock badge; saves from the UI return `409 ENV_LOCKED`.

Two implications:

- Once you decide to manage the schedule from env, keep managing it from env. Removing a var and restarting the container clears the lock; the UI values reappear from `app_config` (where the last UI save was stored, or the defaults if no save ever happened).
- Env-lock is all-or-nothing per tile. Setting just `BACKUP_TIME` locks the whole tile, so `BACKUP_SCHEDULE` and `BACKUP_RETENTION` also stop being editable in the UI even though only one var is set.

## Failure handling

On failure, the error is written to `app_config` under `backup_last_auto_error` and surfaced in the tile. If the admin who owns the box has any push service configured (Apprise, Gotify, ntfy), the scheduler sends a `Backup failed` notification via the shared [push-notify](../integrations/notifications.md) path. Successful runs are silent by design; you get pinged only when something needs your attention.

## Related

- [Backups and restore](backups.md)
- [Environment reference](env-vars.md)
- [Push notifications overview](../integrations/notifications.md)
