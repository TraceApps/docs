# OIDC / SSO overview

Every TraceApps app speaks OpenID Connect out of the box. Set a handful of environment variables, restart, and users see a **Sign in with &lt;your IdP&gt;** button on the login page. This page covers the wire contract, single-provider vs multi-provider, group-to-role mapping, auto-link vs auto-register, and the SSO-only setting. Recipe pages for specific providers (Authentik, Keycloak, and so on) branch off from here.

## Env-var contract

The seeder at `server/lib/oidc-env.js` walks the environment on boot, finds every defined provider, and upserts a row in `oidc_providers`. Providers matched from env are marked with a lock badge in the admin UI and refuse edits (so the environment stays the source of truth).

For the common single-provider case, use the unnumbered aliases:

```env
OIDC_ISSUER=https://auth.example.com
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...changeme...
OIDC_REDIRECT_URIS=https://cook.example.com/api/oidc/callback
OIDC_DISPLAY_NAME=Authentik
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=admins
OIDC_AUTO_LINK=1
OIDC_AUTO_REGISTER=0
OIDC_IS_ACTIVE=1
OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=1
```

Full field reference (defaults in parentheses):

| Variable | Purpose |
| --- | --- |
| `OIDC_ISSUER` | Base issuer URL. The app appends `/.well-known/openid-configuration` for discovery. |
| `OIDC_CLIENT_ID` | Client id registered with your IdP. |
| `OIDC_CLIENT_SECRET` | Required for confidential clients. Public clients (PKCE) can omit it with `OIDC_TOKEN_AUTH_METHOD=none`. |
| `OIDC_DISPLAY_NAME` | Text shown on the login button. Defaults to something derived from the issuer host. |
| `OIDC_LOGO_URL` | Optional logo shown beside the button. |
| `OIDC_SCOPE` | Scopes requested (`openid profile email`). Add extras with a space, for example `openid profile email groups`. |
| `OIDC_REDIRECT_URIS` | Comma-separated list of allowed callback URLs. The first one is used by default. |
| `OIDC_TOKEN_AUTH_METHOD` | `client_secret_post` (default), `client_secret_basic`, or `none`. |
| `OIDC_ADMIN_GROUP_CLAIM` | Name of the claim that carries the user's groups. |
| `OIDC_ADMIN_GROUP_VALUE` | Value inside that claim that promotes the user to admin. |
| `OIDC_AUTO_LINK` | `1` = link to an existing local account by verified email (default). |
| `OIDC_AUTO_REGISTER` | `1` = create a new local account on first SSO login (default `0`). |
| `OIDC_IS_ACTIVE` | `1` = show the provider on the login page (default). Set to `0` to stage a provider without exposing it. |
| `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN` | `0` disables local password login server-wide (SSO-only mode). Unset lets the admin UI toggle it. |

Every server env var also accepts a `<NAME>_FILE` suffix for Docker secrets: `OIDC_CLIENT_SECRET_FILE=/run/secrets/oidc_secret` reads the value from disk instead of the env.

## Single vs multi-provider

Multi-provider works by numbering the prefix: `OIDC_PROVIDER_2_ISSUER`, `OIDC_PROVIDER_2_CLIENT_ID`, and so on for a second IdP. Add `_3_`, `_4_`, no fixed cap. The unnumbered `OIDC_*` aliases are treated as a shortcut for `OIDC_PROVIDER_1_*`, so the two shapes never collide.

```env
# Provider 1: Authentik for staff
OIDC_PROVIDER_1_ISSUER=https://auth.company.com/application/o/cooktrace/
OIDC_PROVIDER_1_CLIENT_ID=cooktrace
OIDC_PROVIDER_1_CLIENT_SECRET=...
OIDC_PROVIDER_1_DISPLAY_NAME=Company SSO
OIDC_PROVIDER_1_ADMIN_GROUP_CLAIM=groups
OIDC_PROVIDER_1_ADMIN_GROUP_VALUE=cooktrace-admins

# Provider 2: Pocket-ID for the family
OIDC_PROVIDER_2_ISSUER=https://pocket.home.arpa
OIDC_PROVIDER_2_CLIENT_ID=cooktrace-home
OIDC_PROVIDER_2_CLIENT_SECRET=...
OIDC_PROVIDER_2_DISPLAY_NAME=Home SSO
OIDC_PROVIDER_2_AUTO_REGISTER=1
```

Providers are keyed on `(issuer_url, client_id)`, so renaming your display name or logo doesn't create duplicates on restart. Removing a provider from env doesn't delete the DB row (there might still be linked users); it just unlocks the row for admin edits again.

## Admin role via IdP groups

Set `ADMIN_GROUP_CLAIM` to the claim name your IdP puts groups in (`groups`, `roles`, `memberOf`, whatever your provider uses) and `ADMIN_GROUP_VALUE` to the specific group that grants admin. On every login the app inspects the ID token, and if that value appears inside that claim, the linked local user gets `role='admin'` for the session. Drop from the group in your IdP, next login the user drops back to a plain member. No sync job, no separate admin toggle to maintain.

Leave both unset if you'd rather manage admin from inside the app. In that case SSO logins are always members unless you promote them manually.

## Auto-link vs auto-register

Two independent knobs govern what happens on a user's first SSO login:

- **`AUTO_LINK` (default on)**: if the ID token's `email` matches an existing local user's email (and the IdP marks it verified, when your IdP supports `email_verified`), the SSO identity is linked to that account. The user keeps their history, favorites, whatever. Turn this off if you'd rather that IdP-driven accounts stay separate from anything created locally.
- **`AUTO_REGISTER` (default off)**: if no local user matches, create a new one on the fly with the IdP's `preferred_username`, `name`, and `email`. Turn this on for household/team setups where inviting each new member by hand is friction. Leave off if you want new members to go through the invite flow.

Both off means SSO logins are only allowed for users an admin has already invited (and where email happens to match), which is the strictest option.

## SSO-only mode

Setting `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0` turns off local password login server-wide. The `/login` endpoint returns `403` for password attempts and the UI hides the username/password form; only the SSO buttons render. Truthy values (`1`, `true`, `yes`, `on`) explicitly enable password login. Unset leaves the admin UI in charge of the toggle.

Read [SSO-only mode and recovery](sso-only.md) before turning this on, or you can lock yourself out.

## Per-app cookie names

One tiny divergence to know about if you're browsing dev tools. Each app writes a scoped logout cookie during the OIDC round-trip: `ct_oidc_logout` (CookTrace), `lt_oidc_logout` (LiftTrace), `nt_oidc_logout` (NutriTrace). This lets all three run on sibling subdomains without stomping on each other's session state.

## Provider recipes

Copy-paste guides per IdP, IdP-side setup plus TraceApps env vars:

- [Authentik](oidc/authentik.md)
- [Keycloak](oidc/keycloak.md)
- [Pocket-ID](oidc/pocket-id.md)
- [Authelia](oidc/authelia.md)
- [Google Workspace](oidc/google.md)
- [Auth0](oidc/auth0.md)

## Related

- [Local users, invites, roles](local-users.md)
- [SSO-only mode and recovery](sso-only.md)
- [Session lifetime and password policy](sessions.md)
