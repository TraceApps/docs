# Local users, invites, roles

The local-user layer is a bcrypt-backed username/password store shared by all three apps. Same rules, same rate-limits, same recovery pattern in CookTrace, LiftTrace, and NutriTrace. This page covers how accounts get created, who can invite whom, what the password policy actually checks, and how to recover if you lock yourself out.

## First user becomes admin

On a brand-new install the account database is empty. The first call to `POST /api/auth/register` is treated as the setup step: whoever registers first is stamped with `role='admin'` regardless of what the request asked for, every row that was written while the instance had no accounts is re-parented to that admin's id, and a session cookie is set. Every subsequent registration requires an admin JWT, and no later registration ever re-parents anything.

Practically that means the person who opens the app the first time after install is the operator. Set a strong password on that account, then invite the rest of the household from Settings.

## Single-user vs multi-user

Every app runs multi-user by default. If you're the only user, you can turn multi-user off from Settings, Users, Disable user management. That endpoint (`DELETE /api/auth/management`) wipes the `users` table and stores `single_user_mode=1` in `app_config` so the setup wizard doesn't trigger on reload. The app then runs as if it were single-tenant: no login screen, everything on one shared dataset. Re-enable it by registering a new account through Settings, which clears the setting automatically.

Single-user mode is a UX choice, not a security posture. The app still binds on the network. Put it behind a reverse proxy with real auth (or don't expose it) if the host is reachable.

## Turning user management on later

This is the common path: run single-user for a while, then decide you want real accounts. Your existing data comes with you.

While an instance has no accounts there is no user id to attribute rows to, so everything is written under a placeholder owner. Creating the first account re-points all of it at that account in one transaction, so the new admin opens the app and finds their history intact. This works the same way whether you create that account with a password or let the first OIDC sign-in bootstrap it. Concretely, that covers:

=== "NutriTrace"

    Diary (including notes, water and body stats), local custom foods, meals and recipes, the Activity log, fasting sessions, Trace chat history, wellness data and workouts pulled from a wearable, and the Fitbit / Google Health / Withings / Garmin connections themselves.

=== "LiftTrace"

    Workout log, body stats, cardio log, custom exercises, custom programs, and Trace chat history. The shared global exercise catalog is left alone, since it belongs to every user rather than to whoever registers first.

=== "CookTrace"

    Recipes, pantry, cook diary, shopping list, cookbooks, your recipe and pantry categories, custom and disabled units, and Trace chat history.

Nothing is asked of you beyond creating the account. There is no export/import step, and you should not need to touch the database.

!!! note "Goals and other preferences live in the browser"
    Per-user preferences (goals, meal names, date of birth, units, theme) are stored in the browser rather than on the server while the instance is in single-user mode, because the settings API has no user to key them against. Enabling user management moves them onto the new account automatically, but only in the browser you do it from. If you enable user management on your laptop, your phone will pick the settings up on its next sign-in from the server, but a device that never signed in will start from defaults. Enable it from the device that has the settings you want to keep.

!!! note "Already enabled it and lost something?"
    Earlier builds only re-pointed part of the data, so an instance that switched over before this was fixed had the rest stranded. Upgrading repairs it: on the first start after the update, an instance with exactly one account adopts anything left unowned and logs what it took. Nothing is guessed at. An instance with no accounts is ordinary single-user mode and is left alone, and one with several accounts has no single rightful owner, so the leftovers are reported in the log rather than handed to whoever happens to be first.

## Turning user management back off

Disabling it (Settings, Users, Disable user management) removes every account. What happens to the data differs by app, and the confirmation dialog in each one states which:

- **NutriTrace and CookTrace** delete the data along with the accounts. The dialog says "their data cannot be recovered", and it means it. Disable then re-enable is not a round trip.
- **LiftTrace** keeps it. Its dialog only offers to remove accounts, so a single account's data is handed back to single-user mode and is waiting for you afterwards, including if you later re-enable. With more than one account there is no single owner to hand it to, so it is removed instead.

In all three, deleting an individual account removes that account's data, including any connected wearable tokens.

!!! warning "Take a backup before experimenting"
    Whichever app you are on, run a Full Backup before turning user management off if you are not certain what you want. It is the only way back from the deleting variants.

## Password policy

Minimum requirements enforced at register, reset, and change:

- 8 characters or more
- at least one lowercase letter
- at least one uppercase letter
- at least one digit
- at least one non-alphanumeric character

If the admin sets `password_policy=strong` in Settings, the same rules apply plus a zxcvbn strength check (`STRONG_MIN_SCORE`, currently 3 out of 4). Weak-but-valid passwords like `Password1!` are rejected under `strong`; a passphrase of three or four uncommon words easily clears it.

## Rate limits

Two independent buckets guard the login endpoint, both with a rolling 15-minute window:

- **Per IP**: 10 attempts, then `429 Too many login attempts`.
- **Per username**: 5 attempts, so an attacker rotating IPs still can't grind one account.

The same limiter also protects `POST /api/auth/forgot-password` and `POST /api/auth/recover`.

## Invites

From the admin's Settings, Users pane, click **Invite user**. Two paths:

- **SMTP configured**: enter the invite email, pick a role (`user` or `admin`), and the app sends a branded invite link (`/#/accept-invite?token=...`). The token is a 32-byte random hex string, valid for 7 days.
- **No SMTP**: the same token is generated but the app returns the URL directly for you to copy and hand over out of band (Signal, chat, whatever). Same 7-day expiry, same accept flow.

Pending invites appear in a list you can revoke individually. Accepting an invite lets the recipient pick their own username and password; the invited email address is pre-attached to the account.

!!! tip "SMTP is optional but useful"
    Without SMTP you can still invite people (copyable link), reset your own password (nobody will email you the link, so you'll need the recovery route below), and log in. Setting up SMTP unlocks all three flows. See [Email / SMTP](../integrations/smtp.md).

## Locked out: `RECOVERY_TOKEN`

If you forget the sole admin password and don't have SMTP configured, there's a way back in without touching the database. Set `RECOVERY_TOKEN` to a long random string in the container's environment, restart, then call `POST /api/auth/recover` with `{ "token": "<value>" }`. The server clears the `users` table, drops back into single-user mode, and lets you register fresh through the wizard.

```env
RECOVERY_TOKEN=$(openssl rand -base64 32)
```

!!! warning "This wipes every user account"
    `POST /api/auth/recover` deletes all rows in `users`. Your food/workout/recipe data survives (it stays attached to the row ids the first fresh account will inherit as admin, same as first-run behavior), but everyone else has to be re-invited afterwards.

Rotate `RECOVERY_TOKEN` back out of the environment after use, or leave it set to a value you keep in a password manager. A missing or empty `RECOVERY_TOKEN` returns `503 Recovery not available` so a curious guest can't try their luck.

## Related

- [OIDC / SSO overview](oidc.md)
- [Session lifetime and password policy](sessions.md)
- [SSO-only mode and recovery](sso-only.md)
- [Email / SMTP](../integrations/smtp.md)
