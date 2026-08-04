# Install with Docker Compose

Docker Compose is the supported install path for CookTrace, LiftTrace, and NutriTrace. One `docker-compose.yml`, one `.env`, `docker compose up -d`, done. All three images are multi-arch (`linux/amd64` and `linux/arm64`) and published to **`ghcr.io/traceapps/<app>`** (primary) and **`traceapps/<app>`** on Docker Hub (mirror). The examples below use GHCR; swap in the Docker Hub short form (e.g. `traceapps/cooktrace:1`) if that fits your environment better.

## Prerequisites

- Docker Engine 24+ with the Compose plugin (`docker compose`, not the legacy `docker-compose`).
- Roughly 200 MB of disk for the container image, plus whatever your data grows to (SQLite database plus uploaded images).
- A long random string for `JWT_SECRET`. Generate one with `openssl rand -base64 48`.

## Minimal compose file

Pick the tab for the app you're installing. Each snippet is a complete, working file. Save it as `docker-compose.yml` in an empty directory and run `docker compose up -d` from there.

=== "CookTrace"

    ```yaml
    services:
      cooktrace:
        image: ghcr.io/traceapps/cooktrace:1
        container_name: cooktrace
        ports:
          - "3003:3001"
        volumes:
          - ./data/db:/data/db
          - ./data/uploads:/data/uploads
        environment:
          DB_PATH: /data/db/cooktrace.db
          UPLOADS_PATH: /data/uploads
          JWT_SECRET: change-me-to-a-long-random-string
        env_file: .env
        restart: unless-stopped
    ```

    Container listens on `3001`, exposed on host port `3003`. Open `http://localhost:3003` after the container is up.

=== "LiftTrace"

    ```yaml
    services:
      lifttrace:
        image: ghcr.io/traceapps/lifttrace:1
        container_name: lifttrace
        ports:
          - "3002:3003"
        volumes:
          - ./data/db:/data/db
          - ./data/uploads:/data/uploads
        environment:
          DB_PATH: /data/db/lifttrace.db
          UPLOADS_PATH: /data/uploads
          JWT_SECRET: change-me-to-a-long-random-string
        restart: unless-stopped
    ```

    Container listens on `3003`, exposed on host port `3002`. Open `http://localhost:3002` after the container is up.

=== "NutriTrace"

    ```yaml
    services:
      nutritrace:
        image: ghcr.io/traceapps/nutritrace:1
        container_name: nutritrace
        ports:
          - "3001:3001"
        volumes:
          - ./data/db:/data/db
          - ./data/uploads:/data/uploads
        environment:
          DB_PATH: /data/db/nutritrace.db
          UPLOADS_PATH: /data/uploads
          JWT_SECRET: change-me-to-a-long-random-string
        env_file: .env
        restart: unless-stopped
    ```

    Container listens on `3001`, exposed on host port `3001` (same number for convenience). Open `http://localhost:3001` after the container is up. NutriTrace's image is `node:20-slim` (Debian) rather than Alpine because DuckDB's node bindings need glibc.

!!! info "Host-port defaults are staggered"
    Defaults are chosen so all three can run on the same host without editing, and all avoid the very common `:3000`: NutriTrace on `3001`, LiftTrace on `3002`, CookTrace on `3003`. If any clash with something else on your host, change the left-hand port value in the mapping (e.g. `"3010:3001"` for CookTrace) or put everything behind a reverse proxy.

## Picking an image tag

Each release publishes to several tags at once. Pin to whatever risk level fits.

| Tag | Updates when | Use case |
|---|---|---|
| `:1.2.3` | Never (exact pin) | Reproducible pin to one release |
| `:1.2` | Any `1.2.x` patch | Auto-receive bug fixes, no new features |
| `:1` | Any `1.x.y` minor | Auto-receive minor releases within the major |
| `:latest` | Every stable release | Whatever the newest stable happens to be |
| `:dev` | Every push to `dev` branch | Leading edge, not for production |

The default compose files above use `:1` because that's the sweet spot for most self-hosters: patch and minor updates land automatically, breaking changes never do. Skittish? Pin to `:1.2` or a full `:1.2.3`. Adventurous? Use `:dev` and read the CHANGELOG before each `pull`.

More detail in [Docker image tag matrix](../reference/image-tags.md).

## Ports and volumes at a glance

| App | Container port | Suggested host port | DB volume | Uploads volume |
|---|---|---|---|---|
| NutriTrace | `3001` | `3001` | `/data/db` | `/data/uploads` |
| LiftTrace | `3003` | `3002` | `/data/db` | `/data/uploads` |
| CookTrace | `3001` | `3003` | `/data/db` | `/data/uploads` |

Both volumes are plain bind mounts. Back them up with the same tool you use for the rest of your host (rsync, restic, borg, whatever).

## Using an `.env` file

Anything past the required trio (`DB_PATH`, `UPLOADS_PATH`, `JWT_SECRET`) is easier to keep out of the compose file. Create `.env` next to `docker-compose.yml`:

```env
JWT_SECRET=paste-openssl-rand-base64-48-output-here
TOKEN_ENC_KEY=paste-another-openssl-rand-base64-48-here
RECOVERY_TOKEN=one-more-random-string

SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=user@example.com
SMTP_PASS=your-smtp-password
SMTP_FROM="TraceApps" <noreply@example.com>
```

CookTrace and NutriTrace include `env_file: .env` in their compose examples so every variable in `.env` reaches the container. LiftTrace works the same way once you add the `env_file:` line, or you can enumerate variables under `environment:`. See [Environment reference](../self-hosting/env-vars.md) for the full list.

For values you don't want on disk in plaintext (SMTP passwords, AI keys, OIDC client secrets), the container also accepts any variable name suffixed with `_FILE`. See [Docker Secrets](../self-hosting/secrets.md).

## Healthcheck

None of the three images include a built-in healthcheck. If you want one, add it yourself:

```yaml
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3001/api/health"]
      interval: 30s
      timeout: 5s
      retries: 3
```

Adjust the port to `3003` for LiftTrace. The `/api/health` endpoint returns `200 OK` with a short JSON body once the app finishes booting.

## First-boot verification

After `docker compose up -d`, tail the container log until it stops moving:

```bash
docker compose logs -f
```

You should see the app announce its listen port, run migrations, then go quiet. Open the app in a browser at the host port from the table above. First screen is the setup wizard, covered in [First-run wizard](first-run.md).

If the browser loads but login fails silently over plain HTTP, jump to [Plain-HTTP LAN (INSECURE_COOKIES)](lan-http.md). That's the single most common first-time issue.

## Related

- [First-run wizard](first-run.md)
- [Plain-HTTP LAN (INSECURE_COOKIES)](lan-http.md)
- [Reverse proxy](reverse-proxy.md)
- [Environment reference](../self-hosting/env-vars.md)
- [Updating](updating.md)
