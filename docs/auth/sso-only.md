# SSO-only mode and recovery

Once OIDC is configured and confirmed working, you can turn off local password login server-wide so users have exactly one way in: the SSO button. This page covers the switch itself, why it's a lockout risk, and the recovery path back.

## Turning off password login

Set the env var:

```env
OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0
```

Restart the container. From that point on:

- The `/login` endpoint returns `403` for username/password attempts.
- The Login page UI hides the username/password form and renders only the SSO buttons.
- `GET /api/auth/status` reports `oidc.enable_email_password_login=false` so mobile clients also know to hide the form.
- The admin UI toggle for password login is disabled (padlock badge), because the env var is now the source of truth.

Truthy values (`1`, `true`, `yes`, `on`) explicitly enable password login. Falsy values (`0`, `false`, `no`, `off`) disable it. Unset leaves the admin UI in charge.

The setting also works for a numbered provider prefix, but there's only one server-wide toggle, so `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN` (no prefix) is the one to use.

!!! danger "This is a lockout risk"
    If OIDC breaks for any reason (your IdP is down, TLS cert expired, discovery URL 404s, admin group misconfigured) you have zero ways in via the browser. The recovery flow below is the only path back. Set up recovery **before** you disable password login, not after.

## Recovery via `RECOVERY_TOKEN`

The one out-of-band way back into an SSO-only instance is the `RECOVERY_TOKEN` env var. When set, it enables a **Locked out?** control on the Login page (right below the SSO buttons) that accepts the token and, on match, wipes user management and drops the instance back into single-user mode. From there you register a fresh admin and reconfigure OIDC.

The full flow:

1. Generate a long random value and add it to your compose `.env`:

    ```env
    RECOVERY_TOKEN=$(openssl rand -base64 32)
    ```

2. Restart the container.
3. On the Login page, click **Locked out?**. A hidden panel expands with a password field.
4. Paste the recovery token, confirm the "this will delete all users" prompt, submit.
5. The server clears the `users` table, sets `single_user_mode=1`, and drops you at the setup wizard. Register a fresh admin.
6. Reconfigure OIDC (env vars survived the wipe, so this is usually just "sign in with your IdP again and the account gets re-linked or re-registered").

Under the hood it's a `POST /api/auth/recover` with `{ "token": "<value>" }`. Same rate limit as the login endpoint (10 attempts per IP, 5 per user per 15-min window) so guessing the token is not a realistic attack.

## Recommended flow

Before you flip `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0` for real:

1. Set `RECOVERY_TOKEN` to a strong random value.
2. Restart, then verify the **Locked out?** button appears on the Login page (do not click it, just check it's there).
3. Save the token in your password manager under this app's entry. This is now your break-glass credential.
4. Set `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0` and restart again.
5. Sign in through the SSO button as your admin account, verify everything still works.

Leave `RECOVERY_TOKEN` set indefinitely. It only does anything when the token in the env matches the token typed on the Login page, so a curious guest hitting the "Locked out?" button can't grief you.

!!! warning "Recovery wipes every user account"
    `POST /api/auth/recover` deletes all rows in `users`. Your data (recipes, workouts, food diary, whatever) survives because it stays attached to its row ids, which the first fresh account will inherit as admin (same as first-run behavior). But everyone else has to be re-invited or re-linked afterwards.

## Related

- [Local users, invites, roles](local-users.md)
- [OIDC / SSO overview](oidc.md)
- [Session lifetime and password policy](sessions.md)
