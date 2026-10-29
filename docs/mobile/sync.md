# How sync works

Sync is the same story in every Trace app by design. `server/routes/sync.js` cross-references NutriTrace's version in the other repos, and the client orchestrator (`src/lib/sync.js`) shares the same shape. This page explains the wire contract, the one rule that settles two edits of the same thing, and what you need to configure as a self-hoster (nothing).

## The wire contract

The Android app talks to two endpoints:

- `POST /api/sync/push` accepts a batch of local changes: `{ tables: { [name]: [row, ...] }, settings: [...] }`. The response is a `{ tables: { [name]: [{ client_id, server_id }] } }` map so the client can rewrite its local IDs. Each row carries the time it was edited, and the app's clock at the push (`client_now`), so the server can set a phone's clock right before it compares times.
- `GET /api/sync/pull?since=<ISO>` returns every row changed at or after the given timestamp, plus a `server_time` value to use as `since` on the next pull.

Both endpoints require a valid session (JWT `Authorization: Bearer` header for the native app).

## Push first, then pull

Every sync round pushes local changes first, waits for the server to acknowledge them, then pulls whatever the server has that is newer. Doing it in that order means the server has already absorbed the client's latest writes before it decides what to return, so the pull never sees a stale view of "what changed" that would trigger a redundant round trip.

The client runs a round on a 30-second background timer and again whenever the app returns to the foreground (`visibilitychange`). In CookTrace (from its next release) the timer runs only while the app is on screen, and a round with nothing new to send or fetch does almost no work. You can force one from Settings if you want an immediate reconciliation.

## Two edits of the same thing: the newer edit of each field

When the same item is changed in two places before they sync (the phone offline and the browser, two phones, the web app's own offline queue), every Trace app settles it with one rule, from each app's next release:

- **Each field keeps its newest edit.** The server keeps, for every synced item, when each of its fields last changed. A note added to a food on your phone and its brand changed on the web both stay; neither copy of the whole item replaces the other.
- **Fields that belong together stay together.** Where fields only make sense side by side, they count as one: a quantity and its unit, a food's nutrients and its serving size, a reminder's time, repeat and time zone. The newer edit brings the whole group, so you never end up with one device's amount and the other's unit.
- **Times are when you made the edit, not when it arrived.** A change made offline on Monday and synced on Wednesday counts as Monday's. A phone whose clock is a few minutes off is set right against the server's clock. Two edits in the same second go to the one arriving last.
- **A delete made before a later edit doesn't win.** If you delete something on one device and later change it on another, it stays. An edit made before the delete doesn't bring it back; one made after it does. A delete and an edit in the same second keep the delete.
- **Saving a form changes only what you edited.** In NutriTrace, NoteTrace and CookTrace, in the browser and the Android app, a save sends the fields you changed, so an edit made elsewhere since you opened the form stays.

Older versions of the apps still sync. They send whole items without saying what changed, so the server counts every field they send as edited at the item's own time. Their edit still loses to a newer one elsewhere, but a newer edit from an older app can put back a field changed on another device. Update every device to get the full rule.

### Where the apps differ

- **NutriTrace.** A day's food entries and water merge entry by entry, so logging on two devices keeps both, and a deleted entry stays deleted. The day's note keeps its newest edit, and each body measurement (weight, body fat, waist) is its own field. A setting changed offline no longer replaces a newer change to the same setting.
- **NoteTrace.** Checklist items merge item by item. When the same note's title or text was changed in both places, the newer stays and the other is kept in [version history](../notetrace/features.md#history), so nothing is lost.
- **CookTrace.** When two devices change the same part of a recipe, the other copy is kept as an [earlier version](../cooktrace/recipes.md#history). CookTrace also tells which fields a device changed from the copy that device last had, so it doesn't depend on the phone's clock for that.
- **LiftTrace.** The Android app doesn't use `/api/sync/push`: it saves through the app's own routes, queuing changes made offline with when they were made and which fields they changed, and the server applies the same rule to each (since 1.4.0). A workout changed in two places merges set by set. `/api/sync/push` remains for other tools: from the next release a pushed row newer than the server's copy is decided field by field, and one older than it changes nothing.

## Inclusive pull boundary

The pull boundary is **inclusive**: rows changed at or after `since` come back, not strict-greater. This is deliberate. SQLite's `datetime('now')` is 1-second precision, so an exclusive `>` boundary silently drops any row written in the same second as the previous `server_time`. The client's upsert is idempotent, so an inclusive boundary is safe: you might see the same row twice across two rounds, but you never lose one.

The pull is picked by when the server last wrote a row, not by the edit time it keeps for the rule above, so an edit made offline days ago still reaches every other device on their next pull.

## Foreign-key translation

When a client creates a new row locally, it generates a `client_id`. The server assigns a `server_id` on push and returns the mapping. Any row that references the new row by foreign key (a diary item that points at a food, a workout that points at an exercise) is rewritten client-side using the returned mapping before it is used again.

The server also plays defense: FK columns are updated with `COALESCE(?, existing)` so a stale mobile payload cannot clobber a newer server-side FK. Push is ordered by dependency (parents before children), and pull is ordered so self-referencing tables come out parents-first.

## Soft-deletes propagate

Deletes are never destructive on the wire. A deleted row keeps its primary key and gets a `deleted_at` timestamp instead of being removed. Both push and pull include soft-deleted rows, so a delete on one device propagates to the other. The client hides them from the UI and prunes them locally after the round completes.

A delete counts as an edit with its own time, so the rule above decides between a delete on one device and an edit on another. When the server keeps the other device's version, the device that sent the delete gets the item back on its next sync.

## What you need to configure

Nothing. Sync is automatic the moment you point the Android app at a server URL and log in. There is no per-server sync tuning knob, no schedule to configure, no push service to wire up. Every self-hosted TraceApps install exposes `/api/sync/*` out of the box.

The only host-side configuration that affects sync is the surrounding setup:

- Your server must be reachable from the phone. On a release-signed APK that means HTTPS (see [HTTPS on the LAN](../self-hosting/lan-https.md)), except NutriTrace 1.5.0 and later, which connect to a server on your own network over plain HTTP once you choose **Connect Anyway** (see [Plain-HTTP LAN](../getting-started/lan-http.md#the-android-apps)).
- Your `JWT_SECRET` must be stable. Rotating it invalidates every existing session, and the phone will need to log in again before sync resumes.
- If you sit behind a reverse proxy, make sure `/api/sync/push` is not body-capped below your batch size (see [Reverse-proxy recipes](../self-hosting/proxy-recipes.md); Cloudflare's free tier caps proxied bodies at 100 MB).

## Debugging

When sync misbehaves, turn on verbose logs and reproduce.

- On the phone: **Settings > Diagnostics**. Turn on the Verbose toggle. Reproduce, then copy or share the diagnostic log.
- On the server: run with `LOG_LEVEL=debug` in the container environment. Push and pull payloads, table counts, and boundary timestamps show up in the container logs.

The client also exposes a `syncState` writable store (`{ syncing, phase, progress, lastSync, error, online }`) that a Diagnostics panel surfaces so you can watch a round in real time.

If the log shows every request refused with 401 instead of a real sync failure, the app's sign-in is no longer valid: sign in again. The plain-HTTP cookie trap in [Troubleshooting](../reference/troubleshooting.md#login-401-loop) is a browser problem; the Android app signs in with a token, not the cookie.

## Related

- [Local vs server-connected mode](modes.md)
- [Install the Android app](install.md)
- [Reverse-proxy recipes](../self-hosting/proxy-recipes.md)
- [Troubleshooting](../reference/troubleshooting.md)
