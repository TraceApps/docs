# Updating

Updating is two commands per app:

```bash
docker compose pull
docker compose up -d
```

Compose pulls the newest image that matches the tag in your `docker-compose.yml`, then recreates the container with the same volumes and env vars. Downtime is a few seconds while the new container starts.

!!! warning "Back up first for major bumps"
    Data lives in your bind-mounted volumes and persists across container recreates, but a bad upgrade is easier to undo when you have a snapshot to fall back to. Before crossing a major version boundary (`1.x` to `2.x`), stop the container, copy the DB and uploads directories aside, then restart and pull. Details at [Backups and restore](../self-hosting/backups.md).

## What "the newest image" means depends on your tag

The image tag in your compose file controls how much you get on each `pull`.

| Your tag | `pull` gives you |
|---|---|
| `:1.2.3` | Nothing new. Exact pins never move. |
| `:1.2` | Any newer `1.2.x` patch release. Bug fixes only, no new features. |
| `:1` | Any newer `1.x.y` release within the same major. New features, no breaking changes. |
| `:latest` | The newest stable release, whatever it is. Crosses majors when a new major is out. |
| `:dev` | Whatever's on the `dev` branch right now. Rebuilt on every push. |

If a `docker compose pull` is unexpectedly quiet and you thought there was a new release, check which tag you're on. `:1.2.3` (an exact pin) never pulls a new image.

## Tag-pinning tactics for skittish operators

For production installs on hardware you don't want to babysit, pin narrower than the defaults suggest:

- `:1.2` receives only patch-level updates. Bug fixes, no behavior changes. Safest for "set it and forget it" boxes.
- `:1.2.3` receives nothing. Use this if you audit each release manually and only move deliberately.

Bump the tag in `docker-compose.yml` on your own schedule, run `docker compose pull && docker compose up -d`, done. Read the [CHANGELOG](../reference/changelogs.md) between bumps so nothing surprises you.

## Rollback with an old tag

Rollback is a tag change and a recreate. Edit `docker-compose.yml`:

```yaml
image: ghcr.io/traceapps/cooktrace:1.2.3   # was :1
```

Then:

```bash
docker compose pull
docker compose up -d
```

The old image is still on GHCR (all tags stay published indefinitely, including legacy `-rc.N` tags from before the semver switch). If migrations ran during the failed upgrade, the schema may be forward of what the old image expects; that's when the pre-upgrade DB snapshot from the backup step above earns its keep. Restore the snapshot before starting the older container.

## After the update

Tail the log until the container quiets down:

```bash
docker compose logs -f
```

Migrations run automatically on startup. Nothing else to do.

## Related

- [Backups and restore](../self-hosting/backups.md)
- [Docker image tag matrix](../reference/image-tags.md)
- [Changelogs](../reference/changelogs.md)
