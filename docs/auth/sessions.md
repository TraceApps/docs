# Session lifetime and password policy

Two independent knobs govern how long a user stays signed in and how strong their password has to be. Both are shared across NutriTrace, LiftTrace, CookTrace and NoteTrace.

## Session lifetime

Sessions are JWTs, delivered as an `HttpOnly` cookie in the browser and as an `Authorization: Bearer` token on the mobile app. Both channels share the same expiry, and both can be signed out before it (see [Signing other devices out](#sign-out-everywhere)).

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

## Signing other devices out { #sign-out-everywhere }

From the next release, every account has a session number, and every session carries the number it was signed in with. The server checks it on every request (the browser's cookie, the app's `Authorization: Bearer` token and LiftTrace's image addresses alike), so a session with an older number stops working at once. The number goes up when:

- the password changes: in Settings > Profile, with an emailed reset link, or by an admin in Settings > Users;
- two-step sign-in is turned on or off, or reset (by an admin, with `RECOVERY_TOKEN`, or with `npm run reset-2fa`);
- someone chooses **Sign Out Everywhere** in Settings > Profile (it asks first).

The device that made the change gets a new session in the same answer and stays signed in. Every other browser, phone and watch is signed out: its next request gets a `401` with `session_ended: true`, and the app goes to the sign-in page, which says it was signed out on this device. Nothing saved offline is lost: the browser's queue and the Android app's unsent changes wait for that account's next sign-in. A Wear OS watch (LiftTrace, CookTrace, NoteTrace) uses its phone's session: when it's refused, the watch keeps what it logged, and when the phone signs in again (or made the change itself) it hands the watch the new session and the watch sends what was waiting.

Sessions signed before this update carry no number and count as 0, the number every account starts with, so the update itself signs nobody out. A changed role (Settings > Users) also takes effect at once, since the role is read from the account rather than the session. A full backup restore keeps each account's number at least where it was, so a restore can't bring a signed-out session back.

API tokens and MCP tokens are separate credentials: none of this signs them out. Revoke them in Settings > API Tokens.

## API tokens { #api-tokens }

From the next release, creating an API token (Settings > API Tokens) takes a fresh check, the same one as linking a sign-in provider: the account's password, and its two-step code (or a recovery code) when that's on. An account that only signs in through single sign-on has nothing to type, so it needs a sign-in within the last 15 minutes: sign out and sign in again first. A session given again by Sign Out Everywhere or a password change keeps the time of its real sign-in, so it never counts as a fresh one.

Each new token gets an expiry: **In 30 Days**, **In 90 Days** (the default), **In 1 Year**, or **No Expiry**. An expired token gets `401 auth_invalid`, like a revoked one. Tokens made before this keep working as they were.

For scripts, `POST /api/admin/api-tokens` takes `name`, `scopes`, `password`, `code` (when two-step sign-in is on) and `expires_in_days` (1 to 3650, or `null` for no expiry).

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

- [Two-step sign-in](two-step.md)
- [Local users, invites, roles](local-users.md)
- [SSO-only mode and recovery](sso-only.md)
- [OIDC / SSO overview](oidc.md)
