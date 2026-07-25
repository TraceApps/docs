# LiftTrace env vars (app-specific)

Only the env vars that are specific to LiftTrace live here. Shared ones (`JWT_SECRET`, `DB_PATH`, `BASE_URL`, `SMTP_*`, `AI_*`, `OIDC_*`, backup schedule, and so on) live in [Shared environment reference](../self-hosting/env-vars.md) since they behave the same across CookTrace, LiftTrace, and NutriTrace.

Every var below also accepts the Docker secrets convention `<NAME>_FILE=/run/secrets/...` handled by the entrypoint. Values are read once at container start; change one and restart the container.

## Reference

| Var | Default | Purpose |
|---|---|---|
| `PORT` | `3003` | Internal Express port. Compose maps host `3002:3003` in the reference file. |
| `EXERCISE_SOURCES` | (empty) | Comma list of exercise-source IDs to auto-seed on first boot: any of `wger`, `free-db`, `exercisedb`, `exercisedb-oss`. Empty triggers the first-run wizard's library picker instead of seeding blind. |
| `EXERCISEDB_OSS_URL` | `https://oss.exercisedb.dev` | Override the community mirror URL for the `exercisedb-oss` source. Point at your own deployment if you want to self-host the mirror or reduce load on the public one. |
| `ALLOW_PRIVATE_RADIO_URLS` | `0` | Set `1` or `true` to let the radio proxy fetch loopback, RFC1918, or IPv6 ULA stream URLs. Off by default to prevent SSRF. A warning is logged at startup when the opt-out is active. |

## Notes on each

### `PORT`

The container listens on `3003`. Every reverse-proxy snippet in the docs assumes that. If you change `PORT` inside the container, keep it in sync with your compose `expose` / `ports` mapping. The healthcheck in the app is `GET /api/health`.

### `EXERCISE_SOURCES`

Auto-seeds one or more exercise catalogs the very first time the DB is created. Handy for provisioning a fresh instance from Ansible or a compose file without walking the wizard. Valid IDs:

- `wger` (~600 exercises, CC-BY-SA 4.0)
- `free-db` (~870 exercises, public domain, images)
- `exercisedb` (~1,300 with GIFs, needs a RapidAPI key configured in Settings)
- `exercisedb-oss` (~1,500 with GIFs, community mirror)

Comma-separated, no spaces required. Empty (the code default) triggers the wizard's library picker so a new admin actively chooses; the README example writes `wger,free-db` which is the safe zero-key seed pair. See [Exercise library](exercises.md#seeding-on-first-boot).

### `EXERCISEDB_OSS_URL`

The `exercisedb-oss` source hits `https://oss.exercisedb.dev` by default. The mirror is community-hosted with no SLA. Point this env at your own deployment (`https://exdb.your-domain.tld`) to reduce load on the public one or to bring the catalog fully in-house. Consumed by `server/exercise-sources/exercisedb-oss.js`.

### `ALLOW_PRIVATE_RADIO_URLS`

The radio proxy blocks stream URLs that resolve to `127.0.0.1`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, or `fc00::/7` by default. Blocked requests fail with:

```text
Private / loopback addresses are blocked. Set ALLOW_PRIVATE_RADIO_URLS=1 to enable LAN streams.
```

If you actually stream from an Icecast box on your LAN and understand the SSRF trade-off, set `ALLOW_PRIVATE_RADIO_URLS=1`. Do not enable on a public-facing deployment unless you have thought carefully about what a hostile station URL could reach on your internal network. Full detail in [Radio player](radio.md#ssrf-safety-allow_private_radio_urls).

## Not LiftTrace-specific

Some vars that live in LT compose files are actually shared. Look them up in [Shared environment reference](../self-hosting/env-vars.md):

- `PORT` alternatives, `NODE_ENV`, `LOG_LEVEL`, `BASE_URL`
- `DB_PATH`, `UPLOADS_PATH`, `BACKUPS_PATH`, `BACKUP_UPLOAD_MAX_MB`
- `JWT_SECRET`, `TOKEN_ENC_KEY`, `INSECURE_COOKIES`, `MAX_SESSION_HOURS`, `RECOVERY_TOKEN`
- `BACKUP_SCHEDULE`, `BACKUP_TIME`, `BACKUP_RETENTION`
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM`
- `AI_PROVIDER`, `AI_API_KEY`, `AI_MODEL`, `AI_BASE_URL`, `AI_ENABLED`
- `OIDC_*` (single-provider plus `OIDC_PROVIDER_2_*` etc.), `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN`

## Related

- [Shared environment reference](../self-hosting/env-vars.md)
- [Settings reference](settings.md)
- [Exercise library](exercises.md)
- [Radio player](radio.md)
