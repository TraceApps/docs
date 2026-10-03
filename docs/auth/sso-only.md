# SSO-only mode and recovery

Once OIDC is configured and confirmed working, you can turn off local password login server-wide so users have exactly one way in: the SSO button. This page covers the switch itself, why it's a lockout risk, and the ways back in.

## Turning off password login

There are two ways to do it:

- **In the app**: the password-login toggle in **Settings → Authentication** (admin only).
- **From the environment**: set the env var, then run `docker compose up -d`:

    ```env
    OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0
    ```

    Truthy values (`1`, `true`, `yes`, `on`) explicitly enable password login, falsy values (`0`, `false`, `no`, `off`) disable it, and unset leaves the toggle in **Settings → Authentication** in charge. While the variable is set, that toggle is locked.

!!! note "The env var needs an env-defined provider"
    In current versions, `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN` only takes effect when at least one provider is also defined through `OIDC_*` env vars. If you added your provider in **Settings → Authentication**, use the toggle there instead.

With password login off:

- Password sign-in attempts get `403`.
- The login page hides the username/password form and shows only the SSO buttons.
- `GET /api/auth/status` reports `oidc.enable_email_password_login=false`, so the Android app hides the form too.

!!! danger "This is a lockout risk"
    If OIDC breaks for any reason (your IdP is down, a TLS certificate expired, the issuer URL changed, the admin group mapping is wrong) nobody can sign in through the browser. Read the next section and prepare a way back **before** you turn password login off.

## Getting back in

### First choice: turn password login back on

Everyone who has a password can sign in again, and nothing is deleted. Accounts that were created through SSO have no password, so make sure at least one admin has one before going SSO-only.

If you turned it off with the env var (and a provider is defined through env vars), set it back and recreate the container:

```env
OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=1
```

```bash
docker compose up -d
```

If you turned it off with the toggle in **Settings → Authentication**, switch it back on from the server with one command. It takes effect immediately, no restart needed. This is for CookTrace; for another app, use its container name and database name (`lifttrace`, `notetrace`, `nutritrace`):

```bash
docker exec cooktrace node -e "const D=require('better-sqlite3'); new D(process.env.DB_PATH || './cooktrace.db').prepare(\"UPDATE app_config SET value='1' WHERE key='enable_email_password_login'\").run()"
```

### Last resort: `RECOVERY_TOKEN`

If no admin can sign in at all, the `RECOVERY_TOKEN` env var enables a **Locked Out?** control on the login page. Entering the token there resets the instance.

!!! danger "Recovery deletes every account and everything they own"
    The reset deletes all accounts, and with them all of their data: recipes, pantry and shopping lists, workouts and programs, food diary, notes, whatever the app holds. It is not a way to recover data. Copy your data folder (the directory mounted at `/data/db`) somewhere safe before using it, and try the first choice above first.

The full flow:

1. Generate a long random value on your machine, and copy the output:

    ```bash
    openssl rand -base64 32
    ```

2. Add it to the app's environment (paste the value itself; `.env` files don't run commands) and run `docker compose up -d`:

    ```env
    RECOVERY_TOKEN=<the value you generated>
    ```

3. On the login page, click **Locked Out?**. A panel with a token field opens.
4. Paste the recovery token, confirm the prompt that all users will be deleted, and submit.
5. The app deletes every account and their data and reopens in single-user mode, with no sign-in. To have accounts again, turn user management back on in **Settings → Users** and create a new admin.
6. Providers defined through env vars are still there, so SSO works again as soon as user management is back on.

Under the hood it's a `POST /api/auth/recover` with `{ "token": "<value>" }`, under the same rate limit as the login endpoint, so guessing the token is not a realistic attack. The control only does anything when the typed token matches the env value, so a curious guest clicking **Locked Out?** can't do any harm.

## Recommended flow

Before you turn password login off for real:

1. Make sure at least one admin account has a password, and that you know it.
2. Confirm SSO works: sign in through the SSO button as that admin.
3. Take a full backup (**Settings → Backup**) and keep it somewhere off the server.
4. Turn password login off (toggle or env var) and sign in through SSO once more to confirm.

If SSO ever breaks, [turn password login back on](#first-choice-turn-password-login-back-on) from the server; nothing is lost that way.

## Related

- [Local users, invites, roles](local-users.md)
- [OIDC / SSO overview](oidc.md)
- [Session lifetime and password policy](sessions.md)
