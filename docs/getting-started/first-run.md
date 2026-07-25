# First-run wizard

The first time you open the app in a browser after `docker compose up -d`, the setup wizard runs. It only appears when there are zero user accounts in the database, so this is a one-time flow per install.

## What the wizard does

The wizard is short by design:

1. Sanity-checks the environment. It confirms the database is writable, `JWT_SECRET` is set to something other than the dev default, and (if you left `INSECURE_COOKIES=0`) that the browser is reaching the app over HTTPS.
2. Prompts for the first admin account (email, display name, password).
3. Creates that user and marks them admin.
4. Signs you in and drops you on the app's home screen.

Every account created after this one is a normal user by default. You can promote another account to admin later from **Settings** then **User Management**.

!!! warning "Set `JWT_SECRET` before you visit the wizard"
    The server refuses to start in production with the built-in `dev-secret`. If you skipped it, the wizard never loads. Set `JWT_SECRET` to a long random string (`openssl rand -base64 48`), `docker compose up -d` again, then reload the page.

## Password policy

Passwords must be at least 8 characters and include an uppercase letter, a lowercase letter, a digit, and a symbol. If you switch the admin **Password policy** setting to `strong`, new passwords are also scored with zxcvbn and low-strength candidates are refused.

## Single-user vs multi-user

All three apps run in "single-user mode" by default. As long as there's exactly one account in the database, the app skips the login screen and auto-signs that user in on every page load. Nobody outside the install sees anything.

The moment you add a second user (from **Settings** then **User Management** then **Add user**, or via an invite email if SMTP is configured), the login screen turns on automatically for every visitor. There's no toggle to flip; the count of users decides.

Delete every non-primary account and the login screen goes away again. That's it.

## Per-app first steps

Each app has its own next-step tour. Do these once and the app is ready to use daily.

=== "CookTrace"

    - Import a first recipe from a URL: paste a link into **Recipes** then **Import**. CookTrace supports 300+ site-specific scrapers.
    - Add a few staples to **Pantry** so the shopping list and cook suggestions have something to work with.
    - Open **Settings** then **Trace AI** if you want conversational recipe search and pantry writes. See [Setting up Trace](../trace/setup.md).

=== "LiftTrace"

    - On first boot the exercise library auto-seeds from wger and a bundled free DB. This takes a minute or two; the **Exercises** page shows a progress bar.
    - Create your first program from **Programs** then **New Program**, or start freestyle from **Diary** and log sets ad hoc.
    - Optional: point the **Radio** player at your Subsonic or Navidrome server from **Settings** then **Radio**.

=== "NutriTrace"

    - Pick your unit preferences and macro targets in the **Onboarding** flow. NutriTrace's Adaptive TDEE learns as you log, so a rough starting point is fine.
    - Log one meal by barcode from the **Diary** to confirm Open Food Facts lookups are reaching the app.
    - Optional: connect a wearable from **Settings** then **Wellness** (Fitbit, Withings, Google Health, Garmin, or Android Health Connect).

## Optional: enable OIDC/SSO later

Local users work fine forever; you don't have to add SSO. When you're ready to hand identity off to Authentik, Keycloak, Pocket-ID, Authelia, Google Workspace, or Auth0, the setup lives in [OIDC / SSO overview](../auth/oidc.md).

Two things worth knowing up front:

- OIDC providers can be defined in env vars or in the admin UI. Env-defined providers are locked from UI edits (with a visible lock badge).
- Setting `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0` puts the app in SSO-only mode. Do this only after you've confirmed at least one admin can sign in via OIDC. Details in [SSO-only mode and recovery](../auth/sso-only.md).

## Related

- [Install with Docker Compose](compose.md)
- [Local users, invites, roles](../auth/local-users.md)
- [Session lifetime and password policy](../auth/sessions.md)
- [Plain-HTTP LAN (INSECURE_COOKIES)](lan-http.md)
