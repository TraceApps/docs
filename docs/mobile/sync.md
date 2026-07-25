# How sync works

Sync is the same story in all three apps by design. `server/routes/sync.js` cross-references NutriTrace's version in the other two repos, and the client orchestrator (`src/lib/sync.js`) shares the same shape. This page explains the wire contract, the conflict rules, and what you need to configure as a self-hoster (nothing).

## The wire contract

The Android app talks to two endpoints:

- `POST /api/sync/push` accepts a batch of local changes: `{ tables: { [name]: [row, ...] }, settings: [...] }`. The response is a `{ tables: { [name]: [{ client_id, server_id }] } }` map so the client can rewrite its local IDs.
- `GET /api/sync/pull?since=<ISO>` returns every row changed at or after the given timestamp, plus a `server_time` value to use as `since` on the next pull.

Both endpoints require a valid session (JWT `Authorization: Bearer` header for the native app).

## Push first, then pull

Every sync round pushes local changes first, waits for the server to acknowledge them, then pulls whatever the server has that is newer. Doing it in that order means the server has already absorbed the client's latest writes before it decides what to return, so the pull never sees a stale view of "what changed" that would trigger a redundant round trip.

The client runs a round on a 30-second background timer and again whenever the app returns to the foreground (`visibilitychange`). You can force one from Settings if you want an immediate reconciliation.

## Last-write-wins, inclusive boundary

Conflict resolution is `updated_at`-based. If the same row exists on both sides, whichever side has the later `updated_at` wins. Every mutable table carries an `updated_at` column and every write bumps it.

The pull boundary is **inclusive**: rows where `updated_at >= since` come back, not strict-greater. This is deliberate. SQLite's `datetime('now')` is 1-second precision, so an exclusive `>` boundary silently drops any row written in the same second as the previous `server_time`. The client's upsert is idempotent, so an inclusive boundary is safe: you might see the same row twice across two rounds, but you never lose one.

## Foreign-key translation

When a client creates a new row locally, it generates a `client_id`. The server assigns a `server_id` on push and returns the mapping. Any row that references the new row by foreign key (a diary item that points at a food, a workout that points at an exercise) is rewritten client-side using the returned mapping before it is used again.

The server also plays defence: FK columns are updated with `COALESCE(?, existing)` so a stale mobile payload cannot clobber a newer server-side FK. Push is ordered by dependency (parents before children), and pull is ordered so self-referencing tables come out parents-first.

## Soft-deletes propagate

Deletes are never destructive on the wire. A deleted row keeps its primary key and gets a `deleted_at` timestamp instead of being removed. Both push and pull include soft-deleted rows, so a delete on one device propagates to the other. The client hides them from the UI and prunes them locally after the round completes.

If a row is edited on one device and deleted on another during the same offline window, last-write-wins on `updated_at` still decides the outcome (the delete counts as a write and bumps `updated_at`).

## What you need to configure

Nothing. Sync is automatic the moment you point the Android app at a server URL and log in. There is no per-server sync tuning knob, no schedule to configure, no push service to wire up. Every self-hosted TraceApps install exposes `/api/sync/*` out of the box.

The only host-side configuration that affects sync is the surrounding setup:

- Your server must be reachable from the phone. On a release-signed APK that means HTTPS (see [HTTPS on the LAN](../self-hosting/lan-https.md)).
- Your `JWT_SECRET` must be stable. Rotating it invalidates every existing session, and the phone will need to log in again before sync resumes.
- If you sit behind a reverse proxy, make sure `/api/sync/push` is not body-capped below your batch size (see [Reverse-proxy recipes](../self-hosting/proxy-recipes.md); Cloudflare's free tier caps proxied bodies at 100 MB).

## Debugging

When sync misbehaves, turn on verbose logs and reproduce.

- On the phone: **Settings > Diagnostics**. Turn on the Verbose toggle. Reproduce, then copy or share the diagnostic log.
- On the server: run with `LOG_LEVEL=debug` in the container environment. Push and pull payloads, table counts, and boundary timestamps show up in the container logs.

The client also exposes a `syncState` writable store (`{ syncing, phase, progress, lastSync, error, online }`) that a Diagnostics panel surfaces so you can watch a round in real time.

If the log shows a 401 loop instead of a real sync failure, check [Troubleshooting](../reference/troubleshooting.md#login-401-loop) for the plain-HTTP cookie trap.

## Related

- [Local vs server-connected mode](modes.md)
- [Install the Android app](install.md)
- [Reverse-proxy recipes](../self-hosting/proxy-recipes.md)
- [Troubleshooting](../reference/troubleshooting.md)
