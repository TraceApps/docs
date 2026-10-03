# Docker image tag matrix

Every app is published to two registries as multi-arch (linux/amd64 + linux/arm64) images. Both registries receive the identical tag set on every build.

**Primary registry (GHCR):**

- `ghcr.io/traceapps/cooktrace`
- `ghcr.io/traceapps/lifttrace`
- `ghcr.io/traceapps/notetrace`
- `ghcr.io/traceapps/nutritrace`

**Mirror (Docker Hub):**

- `traceapps/cooktrace`
- `traceapps/lifttrace`
- `traceapps/notetrace`
- `traceapps/nutritrace`

GHCR is the primary registry. Docker Hub is a discoverability mirror; both are first-class and either works for pulls. Pick whichever your environment prefers. The Docker Hub short form (`traceapps/<app>`) can be handy for quick `docker pull` or when running behind a registry that already caches Docker Hub.

## Tags

Each app publishes two tags:

| Tag | Updates when | Use it for |
|-----|--------------|------------|
| `latest` | Every stable release | Running the app. This is the tag the install guide uses. |
| `dev` | Every push to the `dev` branch | Testing what is coming next; expect rough edges. |

There are no version-number tags such as `:1`, `:1.4` or `:1.4.0`. Version tags were published for the first releases and then retired, because a tag like `:1` quietly stops moving once nothing updates it, and an install on it falls behind without any warning. If your compose file still names one of them, change it to `:latest`.

Tag generation lives in each app's `.github/workflows/docker.yml`: a push to `main` publishes `latest`, a push to `dev` publishes `dev`.

## Pinning one exact build

To hold an install on one build, pin the image by digest instead of a tag. Find the digest you are running:

```bash
docker inspect --format '{{index .RepoDigests 0}}' ghcr.io/traceapps/cooktrace:latest
```

That prints something like `ghcr.io/traceapps/cooktrace@sha256:1bad1f...`. Put that whole value in `image:` and the install stays on that build until you change it. Earlier builds stay on GHCR under their digests, so a digest you noted before an update still pulls for a rollback.

## Pulling and upgrading

```bash
docker compose pull
docker compose up -d
```

On `latest` or `dev`, `pull` fetches the new image and `up -d` recreates the container. On a digest pin, `pull` is a no-op; change the digest in your compose file first, then `pull` and `up -d`.

## Switching between GHCR and Docker Hub

The tag set is identical, so switching registries is a one-line edit in your compose file:

```diff
 services:
   cooktrace:
-    image: ghcr.io/traceapps/cooktrace:latest
+    image: traceapps/cooktrace:latest
```

Then `docker compose pull && docker compose up -d`. Images are byte-for-byte the same build (one push job publishes to both), so container state carries over cleanly.

## Arm64 / Raspberry Pi

Both architectures are in every published image, so a Pi 4 or Pi 5 just pulls the same tag as an x86 server. If you see `no matching manifest for linux/arm64/v8` your Docker install is old; upgrade Docker Engine or Docker Desktop and re-pull.

## Related

- [Install with Docker Compose](../getting-started/compose.md)
- [Updating](../getting-started/updating.md)
- [Release channels](release-channels.md)
- [Changelogs](changelogs.md)
