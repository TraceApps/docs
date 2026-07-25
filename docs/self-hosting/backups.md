# Backups and restore

You have three ways to back up a TraceApps install, and they layer rather than replace each other. Volume snapshots are the ground truth. The in-app Full Backup ZIP is a convenience for moving an install between hosts. The portable JSON export is per-user data with no server config attached. Pick one, or all three, depending on what you are protecting against.

!!! warning "Test your restore"
    A backup you have never restored is not a backup. Once a quarter, take a copy of your production data, restore it into a scratch container on a different host, log in, and confirm you can read your own history. Every self-hoster who loses data does so at the moment they discover the backup they took never actually worked.

## Layer 1: volume snapshot (the real backup)

Everything the app cares about lives in two directories inside the container: `/data/db` (the SQLite database plus its WAL and shared-memory sidecars) and `/data/uploads` (photos, avatars, and the "Full Backup" ZIP archive). Copy those two directories and you have everything.

```bash
docker compose stop cooktrace
cp -a ./data ./data-backup-$(date +%F)
docker compose start cooktrace
```

You can skip the stop on a running install; SQLite in WAL mode is safe to copy live, but the sidecar `.db-wal` and `.db-shm` files must travel with the main `.db` file or the copy will look truncated. If you use a snapshotting filesystem (ZFS, Btrfs, LVM), take the snapshot instead: it captures all three in one instant.

**Restore.** Stop the container, replace the `./data` directory with your backup, start the container. The schema is idempotent, so a slightly newer image on the destination will migrate on boot.

## Layer 2: in-app Full Backup ZIP

Every app has Settings, Backup and Restore for an admin. The button produces a single ZIP containing an SQL dump of the database and a copy of the uploads tree. Behind the scenes this is `GET /api/full-backup/download`, admin-only, and the ZIP lands in `BACKUPS_PATH` (default `<UPLOADS_PATH>/backups`).

You can also schedule it. Set three env vars and the container's built-in scheduler runs at the given time in the container timezone:

```env
BACKUP_SCHEDULE=daily     # off | daily | weekly | monthly
BACKUP_TIME=03:00         # HH:MM
BACKUP_RETENTION=7        # 1..99
```

Setting any of the three env-locks the tile in Settings, so admins can no longer edit the schedule from the UI. Retention prunes oldest ZIPs first. The scheduler stays inside the container: no host cron, no external timer.

**Restore.** Settings, Backup and Restore, Upload backup. Multer caps uploads at `BACKUP_UPLOAD_MAX_MB` (default `512`). Larger archives should be dropped straight into `BACKUPS_PATH` on the host and restored via the pick-existing-backup path in the UI, which sidesteps the multer cap.

!!! warning "Restore is destructive"
    Uploading a backup ZIP replaces the current database and uploads tree in-place. Take a Layer 1 copy first, then restore. There is no undo.

=== "NutriTrace note"
    The optional Open Food Facts local mirror (`OFF_LOCAL_DB`) is **not** included in the Full Backup ZIP. That file is a shared read-only reference (roughly 7 to 8 GB of Parquet) that you can redownload at any time via Settings, OFF Mirror, Refresh. Keeping it out of the ZIP keeps backups small and restore fast. If you have `OFF_LOCAL_ONLY=1` set (air-gap mode), copy `/data/off-mirror` alongside your Layer 1 snapshot; it is not user data, but you'll want it before your container starts refusing lookups.

## Layer 3: portable JSON export

Per-user, not admin-only. Every app has Settings, Data Export where you can download a JSON snapshot of your own diary, workouts, recipes, and settings. The exact scope depends on the app:

- **CookTrace**: recipes, pantry, cook diary, shopping, cookbooks.
- **LiftTrace**: workouts, programs, PRs, body stats.
- **NutriTrace**: food diary, foods, meals, wellness, body-composition history, streaks.

The JSON is documented and stable enough to migrate between instances or to a fresh install. It is **not** a system backup: OIDC providers, SMTP config, admin roles, and other users' data are excluded by design. Treat it as an escape hatch for a single user's data rather than a disaster-recovery mechanism.

## What is not backed up

- The Docker image itself. Redownload from `ghcr.io/traceapps/<app>:<tag>` (see [Compare](compare.md) for the tag matrix).
- `.env` and `docker-compose.yml`. Version-control these separately; they contain secrets or references to them.
- Reverse-proxy config (Caddyfile, nginx.conf, Traefik dynamic files).
- OS-level state: firewall rules, systemd units, Tailscale keys.

Take one final tarball of your compose directory once you have the app working, and put it somewhere off-host. That plus a Layer 1 snapshot is the full survival kit.

## Related

- [Environment reference](env-vars.md)
- [Scheduled backups](scheduled-backups.md)
- [Updating](../getting-started/updating.md)
