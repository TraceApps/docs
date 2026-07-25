# Session lifetime and password policy

Two independent knobs govern how long a user stays signed in and how strong their password has to be. Both are shared across CookTrace, LiftTrace, and NutriTrace.

## Session lifetime

Sessions are JWTs, delivered as an `HttpOnly` cookie in the browser and as an `Authorization: Bearer` token on the mobile app. Both channels share the same expiry.

The server picks the expiry per token like this:

1. If `app_config.session_hours` is set (Settings, Users, Session Duration), use that value, capped at `MAX_SESSION_HOURS`.
2. Otherwise use the default of `8760` hours (1 year).

The relevant env var is:

```env
MAX_SESSION_HOURS=8760
```

`MAX_SESSION_HOURS` is a **ceiling**, not the default. It's the largest value the admin UI is allowed to store. If your threat model demands shorter sessions, lower this: `MAX_SESSION_HOURS=168` caps everyone at 7 days.

The default of one year sounds long, but it's deliberate for two reasons:

- The mobile app pairs the JWT with a per-open biometric prompt (fingerprint/Face ID/Windows Hello). The JWT is effectively a refresh proxy; the real auth gate is the biometric on cold-start.
- Previously the default was 30 days and users hit silent lockouts every month with no security benefit. If biometric is off, an in-app **Sign out on close** setting still works.

Zero or a negative value is treated as "use the max", not "expire immediately".

### Per-user override

Once user management is active, an admin can lower the session duration for the whole instance from **Settings, Users, Session Duration**. The value is written to `app_config.session_hours` and every new token issued after the change picks it up. Existing tokens keep their original expiry (they were signed with it) until they're refreshed at the next login.

## Password policy

At `POST /api/auth/register`, `PATCH /api/auth/change-password`, and `POST /api/auth/reset-password`, the server runs the same rule set:

- 8 characters or more.
- At least one lowercase letter.
- At least one uppercase letter.
- At least one digit.
- At least one non-alphanumeric character.

If any check fails the endpoint returns `400` with a message the client shows inline.

### Optional strong-policy tier

An admin can switch on **strong password policy** at **Settings, Users, Password policy, Strong**. This writes `password_policy=strong` in `app_config` and layers a [zxcvbn](https://github.com/dropbox/zxcvbn) strength estimator on top of the base rules. The passphrase has to score at least `STRONG_MIN_SCORE` (3 out of 4) to pass. Weak-but-valid values like `Password1!` are rejected under `strong`; a three- or four-word uncommon passphrase clears it easily.

The switch takes effect on the next password submission. Existing passwords are not re-validated retroactively, they're only checked when a user changes theirs or resets it.

There is no env-var toggle for the strong policy; it's an admin decision made in the UI.

## Rate limits on login

Two independent buckets guard `POST /api/auth/login` (and the recovery/forgot-password endpoints), both with a rolling 15-minute window:

- **Per IP**: 10 attempts, then `429 Too many login attempts. Try again in a few minutes.`
- **Per username**: 5 attempts, so an attacker rotating IPs still cannot grind one account.

Both buckets are in-memory. A container restart resets them, which is fine for a self-hosted single-instance deploy.

## Related

- [Local users, invites, roles](local-users.md)
- [SSO-only mode and recovery](sso-only.md)
- [OIDC / SSO overview](oidc.md)
