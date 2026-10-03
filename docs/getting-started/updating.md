# Updating

Updating is two commands per app:

```bash
docker compose pull
docker compose up -d
```

Compose pulls the newest image that matches the tag in your `docker-compose.yml`, then recreates the container with the same volumes and env vars. Downtime is a few seconds while the new container starts.

!!! warning "Back up first for major bumps"
    Data lives in your bind-mounted volumes and persists across container recreates, but a bad upgrade is easier to undo when you have a snapshot to fall back to. Before crossing a major version boundary (`1.x` to `2.x`), stop the container, copy the DB and uploads directories aside, then restart and pull. Details at [Backups and restore](../self-hosting/backups.md).

## LiftTrace and CookTrace 1.3.0: Container Port Change

Starting with 1.3.0, LiftTrace and CookTrace listen inside the container on the same port as their sample host port, like NutriTrace and NoteTrace already do. The host port does not change, so bookmarks and the Android app's server address keep working, but the right-hand side of your compose mapping has to.

| App | Before 1.3.0 | 1.3.0 and later |
|---|---|---|
| LiftTrace | `"3002:3003"` | `"3002:3002"` |
| CookTrace | `"3003:3001"` | `"3003:3003"` |

!!! warning "Action needed when you update"
    If you pull 1.3.0 or later (`:latest`, or `:dev` once it carries the change) with the old mapping, the app will not respond. Update the mapping before or right after the pull, then `docker compose up -d`. Also update anything that talks to the container directly on the old port: a reverse proxy on the same Docker network (`lifttrace:3003`, `cooktrace:3001`), a Traefik `loadbalancer.server.port` label, a Cloudflare Tunnel service URL, or a healthcheck. Installs that set `PORT` themselves are not affected.

## Which tag you are on

The image tag in your compose file decides what a `pull` gives you.

| Your tag | `pull` gives you |
|---|---|
| `:latest` | The newest stable release. |
| `:dev` | Whatever is on the `dev` branch right now. Rebuilt on every push. |
| `@sha256:...` (a digest pin) | Nothing new. A digest names one exact build. |

!!! warning "On `:1`, `:1.x` or `:main`?"
    Those tags were published for the first releases and then stopped updating, so an install on one of them has been sitting on an old build without saying so. Change the tag to `:latest`, then `docker compose pull && docker compose up -d`. If you are coming from before 1.3.0, check the port change above first.

If a `docker compose pull` is unexpectedly quiet and you thought there was a new release, check which tag you are on.

## Rollback

Before an update you might want to undo, note the digest you are running:

```bash
docker inspect --format '{{index .RepoDigests 0}}' ghcr.io/traceapps/cooktrace:latest
```

To roll back, put that value in `docker-compose.yml`:

```yaml
image: ghcr.io/traceapps/cooktrace@sha256:...   # was :latest
```

Then:

```bash
docker compose pull
docker compose up -d
```

Earlier builds stay on GHCR under their digests. If migrations ran during the failed upgrade, the schema may be forward of what the old image expects; that's when the pre-upgrade DB snapshot from the backup step above earns its keep. Restore the snapshot before starting the older container.

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
