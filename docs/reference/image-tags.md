# Docker image tag matrix

Every app is published to GHCR as multi-arch (linux/amd64 + linux/arm64) images. Same tag conventions across all three:

- `ghcr.io/traceapps/cooktrace`
- `ghcr.io/traceapps/lifttrace`
- `ghcr.io/traceapps/nutritrace`

## Tag conventions

Each release adds several tags to the same image so you can pin at whatever risk level fits.

| Tag | Example | Updates when | Immutable? |
|-----|---------|--------------|------------|
| `X.Y.Z` | `1.2.3` | Never; points at one exact build | Yes |
| `X.Y`   | `1.2`   | Any `1.2.z` patch release       | No |
| `X`     | `1`     | Any `1.y.z` minor or patch release | No |
| `latest` | `latest` | Every stable release from `main` | No |
| `X.Y.Z-devNN` | `1.2.3-dev01` | Never; points at one milestone dev build | Yes |
| `dev`   | `dev`   | Every push to the `dev` branch  | No |

Tag generation is driven by [`docker/metadata-action`](https://github.com/docker/metadata-action) in each repo's `.github/workflows/docker.yml`. Semver tags come from git tags of the form `v1.2.3`; `latest` and `dev` come from the `main` and `dev` branches respectively. Milestone dev tags of the form `v1.2.3-dev01` publish `:1.2.3-dev01` alongside `:dev`. See [Release channels](release-channels.md) for the full model, including why the iteration number is zero-padded and glued to `dev` without a dot.

## Legacy release-candidate tags

Pre-1.0 releases used `X.Y.Z-rc.N` tags (for example `0.34.0-rc.42`). Those tags remain in the registry indefinitely so anyone pinned to one keeps working. **No new `-rc` tags are cut post-1.0.0**; the versioning ladder is strict `MAJOR.MINOR.PATCH` now. If you are still on an `-rc` tag, move to a rolling tag when convenient.

## Which tag should you pick?

| Use case | Recommended tag | Why |
|----------|-----------------|-----|
| Personal single-user install; you want the newest features | `latest` | One tag, always current stable, no compose edits between releases |
| Family install; you want bug fixes without breaking-change surprises | `X.Y` (e.g. `1.2`) | Auto-receives patches, holds at the current minor |
| "Production-ish" homelab you care about; you want to opt into upgrades | `X.Y.Z` (e.g. `1.2.3`) | Immutable; upgrades are a deliberate compose edit + `pull` + restart |
| Following along with active development | `dev` | Rolls forward on every push to the `dev` branch; expect breakage |
| Air-gapped mirror where you tarball images | `X.Y.Z` | Pinned digest keeps reproducibility across offline transfers |

## Pulling and upgrading

```bash
docker compose pull
docker compose up -d
```

On a rolling tag (`latest`, `X.Y`, `X`, `dev`), `pull` fetches the new image and `up -d` recreates the container. On an immutable tag (`X.Y.Z`), `pull` is a no-op; bump the tag in your compose file first, then `pull` and `up -d`.

## Arm64 / Raspberry Pi

Both architectures are in every published image, so a Pi 4 or Pi 5 just pulls the same tag as an x86 server. If you see `no matching manifest for linux/arm64/v8` your Docker install is old; upgrade Docker Engine or Docker Desktop and re-pull.

## Related

- [Install with Docker Compose](../getting-started/compose.md)
- [Updating](../getting-started/updating.md)
- [Release channels](release-channels.md)
- [Changelogs](changelogs.md)
