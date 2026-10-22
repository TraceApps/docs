# LiftTrace env vars (app-specific)

Only the env vars that are specific to LiftTrace live here. Shared ones (`JWT_SECRET`, `DB_PATH`, `BASE_URL`, `SMTP_*`, `AI_*`, `OIDC_*`, backup schedule, and so on) live in [Shared environment reference](../self-hosting/env-vars.md) since they behave the same across CookTrace, LiftTrace, and NutriTrace.

Every var below also accepts the Docker secrets convention `<NAME>_FILE=/run/secrets/...` handled by the entrypoint. Values are read once at container start; change one and restart the container.

## Reference

| Var | Default | Purpose |
|---|---|---|
| `PORT` | `3002` | Internal Express port. Compose maps host `3002:3002` in the reference file. Was `3003` before 1.3.0. |
| `EXERCISE_SOURCES` | (empty) | Comma list of exercise-source IDs to auto-seed on first boot: any of `wger`, `free-db`, `exercisedb`, `exercisedb-oss`. Empty triggers the first-run wizard's library picker instead of seeding blind. |
| `EXERCISEDB_OSS_URL` | `https://oss.exercisedb.dev` | Override the community mirror URL for the `exercisedb-oss` source. Point at your own deployment if you want to self-host the mirror or reduce load on the public one. |
| `ALLOW_PRIVATE_RADIO_URLS` | `0` | Set `1` or `true` to let every account play radio stations on your own network (loopback, RFC1918, IPv6 ULA). An admin always can. A warning is logged at startup when it is set. |
| `UPDATE_CHECK` | (unset) | `off` (or `0`, `false`, `no`) stops the server asking GitHub about new releases for good, whatever **Settings, Updates** says, and locks that setting. Unset, the setting decides. 1.4.0 and later. |

## Notes on each

### `PORT`

The container listens on `3002` (it was `3003` before 1.3.0; see [Updating](../getting-started/updating.md#lifttrace-and-cooktrace-130-container-port-change)). Every reverse-proxy snippet in the docs assumes that. If you change `PORT` inside the container, keep it in sync with your compose `expose` / `ports` mapping. The healthcheck in the app is `GET /api/health`.

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

An admin can play stations on your own network (`127.0.0.1`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `fc00::/7`); other accounts get "Could not reach that station". Set `ALLOW_PRIVATE_RADIO_URLS=1` to let every account, and a single-user install, play them. Your music server doesn't need it. Full detail in [Radio player](radio.md#stations-on-your-own-network-allow_private_radio_urls).

### `UPDATE_CHECK`

Update checks are off on a fresh install until someone says yes: the setup wizard asks, and an admin can change it later in **Settings, Updates**. An install that was already running before 1.4.0 keeps checking, as it did before. When checks are on, the server asks `api.github.com` for the latest release at most once a day (**Check Now** asks at once), and nothing about you or your data is sent. Browsers never contact GitHub themselves: they ask your server. The Android app asks GitHub directly for a new APK, and only once you said yes on that phone; `UPDATE_CHECK` doesn't reach it. Read in `server/lib/update-check.js`.

## Not LiftTrace-specific

Some vars that live in LT compose files are actually shared. Look them up in [Shared environment reference](../self-hosting/env-vars.md):

- `PORT` alternatives, `NODE_ENV`, `LOG_LEVEL`, `BASE_URL`, `PUBLIC_URL` (1.4.0 and later: the address in links in emails; from the next release also the app's own address when the server refuses changes sent from other sites' pages, see [If changes are refused](../getting-started/reverse-proxy.md#cross-site-refused))
- `DB_PATH`, `UPLOADS_PATH`, `BACKUPS_PATH`, `BACKUP_UPLOAD_MAX_MB`
- `JWT_SECRET`, `TOKEN_ENC_KEY`, `INSECURE_COOKIES`, `FRAME_ANCESTORS`, `MAX_SESSION_HOURS`, `RECOVERY_TOKEN`
- `BACKUP_SCHEDULE`, `BACKUP_TIME`, `BACKUP_RETENTION`
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM`
- `AI_PROVIDER`, `AI_API_KEY`, `AI_MODEL`, `AI_BASE_URL`, `AI_ENABLED`
- `OIDC_*` (single-provider plus `OIDC_PROVIDER_2_*` etc.), `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN`

## Related

- [Shared environment reference](../self-hosting/env-vars.md)
- [Settings reference](settings.md)
- [Exercise library](exercises.md)
- [Radio player](radio.md)
