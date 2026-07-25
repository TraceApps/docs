# Local users, invites, roles

The local-user layer is a bcrypt-backed username/password store shared by all three apps. Same rules, same rate-limits, same recovery pattern in CookTrace, LiftTrace, and NutriTrace. This page covers how accounts get created, who can invite whom, what the password policy actually checks, and how to recover if you lock yourself out.

## First user becomes admin

On a brand-new install the account database is empty. The first call to `POST /api/auth/register` is treated as the setup step: whoever registers first is stamped with `role='admin'` regardless of what the request asked for, all pre-existing rows with `user_id=NULL` are re-parented to that admin's id, and a session cookie is set. Every subsequent registration requires an admin JWT.

Practically that means the person who opens the app the first time after install is the operator. Set a strong password on that account, then invite the rest of the household from Settings.

## Single-user vs multi-user

Every app runs multi-user by default. If you're the only user, you can turn multi-user off from Settings, Users, Disable user management. That endpoint (`DELETE /api/auth/management`) wipes the `users` table and stores `single_user_mode=1` in `app_config` so the setup wizard doesn't trigger on reload. The app then runs as if it were single-tenant: no login screen, everything on one shared dataset. Re-enable it by registering a new account through Settings, which clears the setting automatically.

Single-user mode is a UX choice, not a security posture. The app still binds on the network. Put it behind a reverse proxy with real auth (or don't expose it) if the host is reachable.

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
