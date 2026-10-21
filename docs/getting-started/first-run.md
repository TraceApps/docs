# First-run wizard

The first time you open the app in a browser after `docker compose up -d`, the setup wizard runs. It only appears when there are zero user accounts in the database, so this is a one-time flow per install.

## What the wizard does

The wizard is short by design:

1. Checks that signing in will stick. Opened at a plain `http://` address other than `localhost` with `INSECURE_COOKIES` off, the account step says **Signing in won't work on this address** and links to [Plain-HTTP LAN](lan-http.md) (NutriTrace 1.4.0, NoteTrace 1.1.0 and LiftTrace 1.4.0 and later; CookTrace from its next release).
2. Prompts for the first admin account (email, display name, password).
3. Creates that user and marks them admin.
4. Signs you in and drops you on the app's home screen.

Every account created after this one is a normal user by default. You can promote another account to admin later from **Settings** then **User Management**.

!!! warning "Set `JWT_SECRET` before you visit the wizard"
    The server refuses to start in production with the built-in `dev-secret`. If you skipped it, the wizard never loads. Set `JWT_SECRET` to a long random string (`openssl rand -base64 48`), `docker compose up -d` again, then reload the page.

## Password policy

Passwords must be at least 8 characters and include an uppercase letter, a lowercase letter, a digit, and a symbol. If you switch the admin **Password policy** setting to `strong`, new passwords are also scored with zxcvbn and low-strength candidates are refused.

## Single-user vs multi-user

CookTrace, LiftTrace, and NutriTrace run in "single-user mode" by default. As long as there's exactly one account in the database, the app skips the login screen and auto-signs that user in on every page load. Nobody outside the install sees anything.

The moment you add a second user (from **Settings** then **User Management** then **Add user**, or via an invite email if SMTP is configured), the login screen turns on automatically for every visitor. There's no toggle to flip; the count of users decides.

Delete every non-primary account and the login screen goes away again. That's it.

!!! note "Two things get called single-user mode"
    The above is the **one-account** case: an account exists, and the app auto-signs it in.
    There is also a **no-accounts** case, reached from Settings, Users, Disable user
    management, where the `users` table is empty and everything is written without an owner.
    They look the same from the outside but behave differently underneath. If you are running
    with no accounts and later want real ones, see
    [Turning user management on later](../auth/local-users.md#turning-user-management-on-later)
    for what carries over.

## Per-app first steps

Each app has its own next-step tour. Do these once and the app is ready to use daily.

=== "CookTrace"

    - Setup asks **Who Do You Cook For?**: the people in your household, with their allergies, diets and the days they're home. You can skip it and fill it in later under [Settings > Household](../cooktrace/settings.md#household-per-user).
    - Import a first recipe from a URL: paste a link into **Recipes** then **Import**. CookTrace supports 300+ site-specific scrapers.
    - Add a few staples to **Pantry** so the shopping list and cook suggestions have something to work with.
    - Open **Settings** then **Trace AI** if you want conversational recipe search and pantry writes. See [Setting up Trace](../trace/setup.md).

=== "LiftTrace"

    - On first boot the exercise library auto-seeds from wger and a bundled free DB. This takes a minute or two; the **Exercises** page shows a progress bar.
    - Create your first program from **Programs** then **New Program**, or start freestyle from **Diary** and log sets ad hoc.
    - Optional: point the **Radio** player at your Subsonic or Navidrome server from **Settings** then **Radio**.
    - Setup asks whether to check for updates (LiftTrace 1.4.0 and later). Checks stay off unless you turn them on, so a fresh install contacts nothing on its own. When on, the web app asks your server, which asks GitHub; the Android app asks GitHub itself, and only once you said yes on that phone. Change it any time in **Settings** then **Updates**, or turn the server's checks off for good with `UPDATE_CHECK=off` (see [LiftTrace env vars](../lifttrace/env-vars.md#update_check)).

=== "NoteTrace"

    - Write a first note from **Capture a thought** at the top of **Notes**, or press **+** for a checklist, a voice note or a picture.
    - Install the Android app and connect it to your server, or install the web app from your browser's menu. Both keep working with no connection.
    - Optional: link your CookTrace shopping list from **Settings** then **CookTrace**. See [CookTrace shopping list](../notetrace/cooktrace.md).
    - Setup asks whether to check for updates (NoteTrace 1.1.0 and later). Checks stay off unless you turn them on, so a fresh install contacts nothing on its own. When on, the web app asks your server, which asks GitHub; the Android app asks GitHub itself, and only once you said yes on that phone. Change it any time in **Settings** then **Updates**, or turn the server's checks off for good with `UPDATE_CHECK=off` (see [NoteTrace env vars](../notetrace/env-vars.md#container)).

=== "NutriTrace"

    - Pick your unit preferences and macro targets in the **Onboarding** flow. NutriTrace's Adaptive TDEE learns as you log, so a rough starting point is fine.
    - Log one meal by barcode from the **Diary** to confirm Open Food Facts lookups are reaching the app.
    - Optional: connect a wearable from **Settings** then **Wellness** (Fitbit, Withings, Google Health, Garmin, or Android Health Connect).
    - Setup asks whether to check for updates (NutriTrace 1.4.0 and later). Checks stay off unless you turn them on, so a fresh install contacts nothing on its own. When on, the web app asks your server, which asks GitHub; the Android app asks GitHub itself, and only once you said yes on that phone. Change it any time in **Settings** then **Updates**, or turn the server's checks off for good with `UPDATE_CHECK=off` (see [NutriTrace env vars](../nutritrace/env-vars.md#update-checks)).

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
