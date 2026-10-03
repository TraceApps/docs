# OIDC / SSO overview

Every TraceApps app speaks OpenID Connect out of the box. Define a provider, either with a handful of environment variables or in **Settings → Authentication**, and users see a **Sign In with &lt;your IdP&gt;** button on the login page. This page covers the callback URL, the env-var contract, single-provider vs multi-provider, group-to-role mapping, auto-link vs auto-register, and the SSO-only setting. Recipe pages for specific providers (Authentik, Keycloak, and so on) branch off from here.

## The callback URL

Every provider needs one callback (redirect) URL, registered in two places: in your IdP, and in the app (`OIDC_REDIRECT_URIS`, or the provider's Redirect URIs in Settings). The two must match exactly. The shape is:

```text
https://<your-host>/api/auth/oidc/callback/<provider-id>
```

- `<your-host>` is the address people open the app at, as the browser sees it. Behind a reverse proxy that is the proxy's public address, not the container's.
- `<provider-id>` is the provider's ID in the app's database. A single provider defined through env vars is almost always `1`. The container log prints the IDs on startup: `[oidc-env] Loaded 1 OIDC provider from environment (IDs: 1)`. A provider added in **Settings → Authentication** gets its callback filled in when you save it; reopen the provider to copy the final URL for your IdP.
- If you serve the app under a subpath with `BASE_URL`, the subpath comes before `/api`: `https://example.com/cooktrace/api/auth/oidc/callback/1`.

The Android app signs in through the same callback, so it needs nothing extra for sign-in. See [Sign-out](#sign-out) for the one Android-specific entry.

!!! warning "A wrong callback path shows a blank page"
    The app also accepts the callback without the number, and at `/api/oidc/callback` (a form older versions of these docs showed). If the IdP sends you back to any other path, the sign-in at the IdP succeeds but the app never receives it, and you land on a blank page. Check the path first when SSO "works at the IdP" but ends on a white screen.

## Env-var contract

On startup each app reads the environment, finds every defined provider, and creates or updates it in the database. Providers defined through env show an **env** badge with a padlock in **Settings → Authentication** and can't be edited there, so the environment stays the source of truth.

Env vars reach the container only if your compose file passes them: list them under `environment:`, or keep them in `.env` with `env_file: .env` on the service (the [install examples](../getting-started/compose.md) include that line). After changing any of them, run `docker compose up -d`; a plain `docker compose restart` keeps the old values.

For the common single-provider case, use the unnumbered aliases:

```env
OIDC_ISSUER=https://auth.example.com/application/o/cooktrace/
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...changeme...
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_DISPLAY_NAME=Authentik
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=admins
OIDC_AUTO_LINK=1
OIDC_AUTO_REGISTER=0
OIDC_IS_ACTIVE=1
```

Full field reference (defaults in parentheses):

| Variable | Purpose |
| --- | --- |
| `OIDC_ISSUER` | Issuer URL. The app fetches `<issuer>/.well-known/openid-configuration` for discovery; a trailing slash works either way. |
| `OIDC_CLIENT_ID` | Client ID registered with your IdP. |
| `OIDC_CLIENT_SECRET` | Required for confidential clients. Public clients can omit it with `OIDC_TOKEN_AUTH_METHOD=none`. Stored encrypted. |
| `OIDC_DISPLAY_NAME` | Text on the login button (`OIDC`, so the button reads **Sign In with OIDC**). |
| `OIDC_LOGO_URL` | Optional logo shown on the login button. |
| `OIDC_SCOPE` | Scopes requested (`openid profile email`). Add extras with a space, for example `openid profile email groups`. |
| `OIDC_REDIRECT_URIS` | Comma-separated callback URLs (see [The callback URL](#the-callback-url)). When there are several, the one on the same address the user opened the app at is used, otherwise the first. |
| `OIDC_TOKEN_AUTH_METHOD` | `client_secret_post` (default), `client_secret_basic`, or `none`. |
| `OIDC_ADMIN_GROUP_CLAIM` | Name of the ID-token claim that carries the user's groups (unset: no group mapping). |
| `OIDC_ADMIN_GROUP_VALUE` | Value inside that claim that makes the user an admin (unset: no group mapping). |
| `OIDC_AUTO_LINK` | `1` = link to an existing local account with the same email, when the IdP marks it verified (default `1`). |
| `OIDC_AUTO_REGISTER` | `1` = create a new local account on first SSO sign-in (default `0`). |
| `OIDC_IS_ACTIVE` | `1` = show the provider on the login page (default). Set to `0` to stage a provider without exposing it. |
| `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN` | `0` disables local password login server-wide (SSO-only mode). Unset lets the admin UI toggle it. See [SSO-only mode](sso-only.md). |

Every server env var also accepts a `<NAME>_FILE` suffix for Docker secrets: `OIDC_CLIENT_SECRET_FILE=/run/secrets/oidc_secret` reads the value from that file instead.

Sign-in uses the authorization code flow with PKCE. The app reads the user's details (`sub`, `email`, `email_verified`, `preferred_username`, `name`, and the group claim) from the ID token, and fills in anything missing there from the IdP's userinfo endpoint (Authelia 4.39 and later only puts them there).

## Single vs multi-provider

Multi-provider works by numbering the prefix: `OIDC_PROVIDER_2_ISSUER`, `OIDC_PROVIDER_2_CLIENT_ID`, and so on for a second IdP. Add `_3_`, `_4_`, no fixed cap. The unnumbered `OIDC_*` aliases are a shortcut for `OIDC_PROVIDER_1_*`; if both are set, the numbered one wins.

```env
# Provider 1: Authentik for staff
OIDC_PROVIDER_1_ISSUER=https://auth.company.com/application/o/cooktrace/
OIDC_PROVIDER_1_CLIENT_ID=cooktrace
OIDC_PROVIDER_1_CLIENT_SECRET=...
OIDC_PROVIDER_1_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_PROVIDER_1_DISPLAY_NAME=Company SSO
OIDC_PROVIDER_1_ADMIN_GROUP_CLAIM=groups
OIDC_PROVIDER_1_ADMIN_GROUP_VALUE=cooktrace-admins

# Provider 2: Pocket ID for the family
OIDC_PROVIDER_2_ISSUER=https://pocket.home.arpa
OIDC_PROVIDER_2_CLIENT_ID=cooktrace-home
OIDC_PROVIDER_2_CLIENT_SECRET=...
OIDC_PROVIDER_2_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/2
OIDC_PROVIDER_2_DISPLAY_NAME=Home SSO
OIDC_PROVIDER_2_AUTO_REGISTER=1
```

The number in `OIDC_PROVIDER_<N>_` and the provider ID in the callback URL are not the same thing: the ID is assigned by the database the first time a provider is seen. On a fresh install they usually line up, but always take the ID from the startup log line (or from **Settings → Authentication**) before registering the callback.

Providers are keyed on `(issuer, client ID)`, so changing the display name or logo doesn't create duplicates, and the ID stays the same across restarts. Removing a provider from env doesn't delete it (there may be linked users); it stays on the login page and becomes editable in **Settings → Authentication**, where you can disable or delete it.

## Admin role via IdP groups

Set `ADMIN_GROUP_CLAIM` to the claim name your IdP puts groups in (`groups`, `roles`, or a namespaced name like `https://example.com/roles`) and `ADMIN_GROUP_VALUE` to the group that grants admin. On every SSO sign-in the app checks the ID token: if the value is in that claim, the account becomes an admin; if it isn't, the account becomes a regular user. Remove someone from the group in your IdP and they drop back to a regular user at their next sign-in. No sync job, no separate admin toggle to maintain.

!!! warning "It demotes as well as promotes"
    With both values set, every SSO sign-in by someone outside the group makes them a regular user, including an admin you created locally and then linked to SSO. Put your own account in the admin group before turning this on.

Leave both unset if you'd rather manage admin from inside the app. SSO sign-ins then never change anyone's role.

## Auto-link vs auto-register

Two independent settings govern what happens the first time someone signs in through a provider:

- **`AUTO_LINK` (default on)**: if the ID token's `email` matches an existing local account and the token says `email_verified: true`, the SSO identity is linked to that account and the person is signed in. Without `email_verified: true` there is no automatic link, and the sign-in is refused with a message asking the person to link the provider from their profile, whatever auto-register is set to. Some IdPs mark every email unverified by default (Authentik since 2025.10, Pocket ID unless configured); the recipe pages show how to change that. With auto-link off, a matching email is refused with a message asking the person to sign in with their password once and link the provider from their profile (**Linked Accounts**); after that, SSO signs them straight in.
- **`AUTO_REGISTER` (default off)**: if no local account matches, create one on the fly. The username comes from `preferred_username` (or the part of the email before the `@`), plus the IdP's `name` and `email`. New accounts are regular users unless the group mapping above makes them admin (or the instance had no accounts yet, in which case the first one becomes the admin). With auto-register off, someone without an account is refused and asked to get an invite first.

The usual setups:

- **Account first, then SSO** (defaults): the person gets an account the usual way (an admin creates it, or they accept an invite), with the same email they have at the IdP. From then on, signing in through SSO links to it automatically, as long as the IdP marks that email verified (see above). A pending invite is not an account yet, so SSO can't link to it until it's accepted.
- **Open to everyone at the IdP**: turn on `AUTO_REGISTER`, and anyone your IdP lets through gets an account.
- **Strictest**: both off. Only people who signed in with a password and linked the provider themselves can use SSO.

## Sign-out

Signing out of the app also ends the session at your IdP when the IdP supports it (RP-initiated logout), so the next sign-in asks for credentials again instead of passing straight through. The app sends the IdP one of these as the place to return to:

- the app's root, for example `https://cook.example.com/` (web), or
- `<app>://oidc-callback` (Android app): `cooktrace://oidc-callback`, `lifttrace://oidc-callback`, `notetrace://oidc-callback` or `nutritrace://oidc-callback`.

If your IdP keeps an allowlist of post-logout redirect URIs (Keycloak does, for example), add both. These belong in the IdP's post-logout list only, not in `OIDC_REDIRECT_URIS`.

## SSO-only mode

Setting `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN=0` turns off local password login server-wide. Password sign-in attempts get `403` and the login page shows only the SSO buttons. Truthy values (`1`, `true`, `yes`, `on`) explicitly enable password login. Unset leaves the admin UI in charge.

Read [SSO-only mode and recovery](sso-only.md) before turning this on, or you can lock yourself out.

## Per-app cookie names

One tiny divergence to know about if you're browsing dev tools. Each app writes a scoped logout cookie during the OIDC round-trip: `ct_oidc_logout` (CookTrace), `lt_oidc_logout` (LiftTrace), `note_oidc_logout` (NoteTrace), `nt_oidc_logout` (NutriTrace). This lets them all run on sibling subdomains without stomping on each other's session state.

## Provider recipes

Copy-paste guides per IdP, IdP-side setup plus TraceApps env vars:

- [Authentik](oidc/authentik.md)
- [Keycloak](oidc/keycloak.md)
- [Pocket ID](oidc/pocket-id.md)
- [Authelia](oidc/authelia.md)
- [Google](oidc/google.md)
- [Auth0](oidc/auth0.md)

## Related

- [Local users, invites, roles](local-users.md)
- [SSO-only mode and recovery](sso-only.md)
- [Session lifetime and password policy](sessions.md)
