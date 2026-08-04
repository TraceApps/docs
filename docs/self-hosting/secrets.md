# Docker Secrets (`*_FILE`)

Every env var the three apps read supports a `<NAME>_FILE` variant that points at a file on disk instead of an inline value. The server reads the file at boot, trims one trailing newline, and treats the contents as if you had set `<NAME>` directly. It is the pattern Postgres, Redis, and a lot of other well-behaved container images use, and it is the cleanest way to keep secrets out of `docker-compose.yml`, `.env`, `docker inspect` output, and shell history.

## When to reach for it

Three concrete cases:

- **Docker Swarm secrets.** Swarm mounts secrets as files under `/run/secrets/<name>`. `<NAME>_FILE` is the direct way to consume them.
- **Bind-mounted secret files.** Handy on a single-host Compose install where you already keep sensitive material in a `secrets/` directory managed by `sops` or `git-crypt` and do not want the plaintext to touch `.env`.
- **Rotating a value without editing compose.** Change the file on disk, restart the container, done. No YAML diff, no risk of committing a real key to git.

Setting both `<NAME>` and `<NAME>_FILE` at the same time is a boot-time error. Pick one.

## Common examples

```env
JWT_SECRET_FILE=/run/secrets/jwt
TOKEN_ENC_KEY_FILE=/run/secrets/token_enc
SMTP_PASS_FILE=/run/secrets/smtp_pass
AI_API_KEY_FILE=/run/secrets/ai_key
OIDC_CLIENT_SECRET_FILE=/run/secrets/oidc_client_secret
```

The suffix works for every server-read env var in every app. That includes `RECOVERY_TOKEN_FILE`, per-provider OIDC secrets (`OIDC_PROVIDER_2_CLIENT_SECRET_FILE`), and anything else in the [Environment reference](env-vars.md).

## Docker Swarm compose stanza

Full worked example for a Swarm stack. The top-level `secrets:` block declares two secrets sourced from disk; the service block mounts them and points the app at `/run/secrets/*`.

```yaml
version: "3.9"

secrets:
  jwt:
    file: ./secrets/jwt.txt
  smtp_pass:
    file: ./secrets/smtp_pass.txt

services:
  cooktrace:
    image: ghcr.io/traceapps/cooktrace:1
    environment:
      NODE_ENV: production
      JWT_SECRET_FILE: /run/secrets/jwt
      SMTP_HOST: smtp.example.com
      SMTP_USER: cooktrace@example.com
      SMTP_PASS_FILE: /run/secrets/smtp_pass
    secrets:
      - jwt
      - smtp_pass
    volumes:
      - ./data/db:/data/db
      - ./data/uploads:/data/uploads
    ports:
      - "3001:3001"
```

Same shape works for LiftTrace (`ghcr.io/traceapps/lifttrace:1`, container port 3003) and NutriTrace (`ghcr.io/traceapps/nutritrace:1`, container port 3001).

## Gotchas

- The file must be readable by the container process. All three images run as a non-root `node` user; if you keep the file `0400 root:root` on the host, the container cannot read it.
- One trailing newline is trimmed; the rest of the file is used verbatim. A leading blank line will end up in the value.
- Docker Compose outside Swarm mode also supports the `secrets:` block with `file:` sources. Swarm is not required to use `<NAME>_FILE`.

## Related

- [Environment reference](env-vars.md)
- [Backups and restore](backups.md)
