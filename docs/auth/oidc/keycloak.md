# OIDC recipe: Keycloak

Keycloak is the reference-implementation OIDC server. If you already run it for other apps, TraceApps drops in as another client. This recipe assumes a Keycloak instance you can log into with realm-admin rights, and was checked against Keycloak 26.8. Adjust hostnames and realm names to match; the examples use CookTrace at `https://cook.example.com`.

## Keycloak side

### 1. Pick or create a Realm

Log into the Keycloak admin console. If you have a shared realm for internal apps (`main`, `home`, whatever), use that. Otherwise **Create Realm**, name it something like `home`, save.

### 2. Create a Client

**Clients, Create client**.

- **Client type**: `OpenID Connect`.
- **Client ID**: `cooktrace` (or `lifttrace`, `notetrace`, `nutritrace`). Use the same string in your env vars.
- **Name**: `CookTrace`.
- Next.
- **Client authentication**: `On` (confidential client).
- **Authorization**: leave off.
- **Standard flow**: on. Leave everything else off (Direct access grants, Implicit flow, Service account roles, the token exchange, device and CIBA grants).
- **Require PKCE**: on, with method `S256` (the app always sends a PKCE challenge). Older Keycloak versions without this switch work too.
- Next.
- **Root URL**: `https://cook.example.com`.
- **Home URL**: `https://cook.example.com/`.
- **Valid redirect URIs**: `https://cook.example.com/api/auth/oidc/callback/1`. The `1` is the provider ID in the app; see [The callback URL](../oidc.md#the-callback-url).
- **Valid post logout redirect URIs**: `https://cook.example.com/`, plus `cooktrace://oidc-callback` if you use the Android app. Signing out of the app also ends the Keycloak session and returns here. If you leave this empty, Keycloak only accepts the redirect URIs above, which don't include these, so the return after sign-out fails.
- **Web origins**: `https://cook.example.com` (or `+` to inherit from redirect URIs).
- Save.

Open the new client, go to **Credentials**, copy the **Client secret**. That value goes into `OIDC_CLIENT_SECRET`.

### 3. Note the issuer URL

Keycloak's issuer URL always includes the realm name:

```
https://keycloak.example.com/realms/home
```

Confirm by hitting `/.well-known/openid-configuration` under that path and checking the `issuer` field matches exactly. That's what goes into `OIDC_ISSUER`.

### 4. (Optional) Group claim mapper

Keycloak doesn't put groups in the ID token by default. To use `OIDC_ADMIN_GROUP_CLAIM` for admin promotion, add a mapper.

In your client, **Client scopes**, click `cooktrace-dedicated` (the per-client scope Keycloak auto-created), then **Add mapper, By configuration**, pick **Group Membership**.

- **Name**: `groups`.
- **Token Claim Name**: `groups`.
- **Full group path**: off. It's on by default, which makes the claim carry `/cooktrace-admins` instead of `cooktrace-admins`.
- **Add to ID token**: on.
- **Add to access token**: on.
- **Add to userinfo**: on.

Save. Then **Groups, Create group**: `cooktrace-admins`. Add your admin user to it (**Users, &lt;user&gt;, Groups, Join group**).

### 5. Verified emails

Keycloak sends `email_verified` from each user's **Email verified** flag. The app only links an SSO sign-in to an existing account in the app when that flag is on, so for people who already have an account, turn it on (**Users → &lt;user&gt; → Email verified**), or have them link Keycloak from their profile (**Linked Accounts**) after signing in with their password. Without either, their first SSO sign-in is refused, or with `OIDC_AUTO_REGISTER=1` gets a second, empty account.

## TraceApps side

Add to the app's environment, for example in `.env` next to the compose file (the [install examples](../../getting-started/compose.md) load it with `env_file: .env`):

```env
OIDC_ISSUER=https://keycloak.example.com/realms/home
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...paste from Credentials tab...
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_DISPLAY_NAME=Keycloak
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

If you set up the group mapper in step 4, add:

```env
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=cooktrace-admins
```

For self-service registration (matches how you'd run this at home for family accounts):

```env
OIDC_AUTO_REGISTER=1
```

Recreate the container (a plain restart keeps the old values):

```bash
docker compose up -d
```

The log should show `[oidc-env] Loaded 1 OIDC provider from environment (IDs: 1)`. If the ID isn't `1`, use that number in the callback URL, in both Keycloak and `OIDC_REDIRECT_URIS`.

## Verify

1. Load the app in a private window. A **Sign In with Keycloak** button appears next to the local login form.
2. Click. Keycloak redirects to its login page, you sign in, and you land back inside the app, signed in.
3. In **Settings → Authentication** the provider row is present with an **env** padlock badge.
4. If you set up the group mapper, an account in `cooktrace-admins` shows as an admin in **Settings → Users** after its next SSO sign-in.

## Troubleshooting

!!! warning "Blank page after signing in at Keycloak"
    The redirect URI is wrong, so Keycloak sent you back to a path the app doesn't handle. It must be exactly `https://<your-host>/api/auth/oidc/callback/<id>`, the same in Keycloak and in `OIDC_REDIRECT_URIS`.

!!! warning "Issuer must include `/realms/<name>`"
    Every Keycloak realm has its own issuer path. Setting `OIDC_ISSUER=https://keycloak.example.com` (without `/realms/home`) will fail discovery with a `404`. Copy the exact value from `/.well-known/openid-configuration` under the realm.

!!! warning "Group claim missing after login"
    If the `groups` mapper is only added to the access token, the ID token TraceApps reads won't carry it. Toggle **Add to ID token** on. Log in again in a fresh session (existing sessions cache the old ID token).

!!! tip "Public clients"
    If you want a PKCE-only setup (no client secret), turn **Client authentication** off in step 2 and set `OIDC_TOKEN_AUTH_METHOD=none` in the env. Omit `OIDC_CLIENT_SECRET`. Confidential is preferred for a server-side app like TraceApps.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Authentik recipe](authentik.md)
