# OIDC recipe: Keycloak

Keycloak is the reference-implementation OIDC server. If you already run it for other apps, TraceApps drops in as another client. This recipe assumes a Keycloak instance you can log into with realm-admin rights. Adjust hostnames and realm names to match.

## Keycloak side

### 1. Pick or create a Realm

Log into the Keycloak admin console. If you have a shared realm for internal apps (`main`, `home`, whatever), use that. Otherwise **Create Realm**, name it something like `home`, save.

### 2. Create a Client

**Clients, Create client**.

- **Client type**: `OpenID Connect`.
- **Client ID**: `cooktrace` (or `lifttrace`, `nutritrace`). Use the same string in your env vars.
- **Name**: `CookTrace`.
- Next.
- **Client authentication**: `On` (confidential client).
- **Authorization**: leave off.
- **Standard flow**: on. **Direct access grants**: off. **Service accounts**: off. **OAuth 2.0 device authorization**: off.
- Next.
- **Root URL**: `https://cook.example.com`.
- **Home URL**: `https://cook.example.com/`.
- **Valid redirect URIs**: `https://cook.example.com/api/oidc/callback`.
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
- **Full group path**: off (so the claim carries just `admins`, not `/admins`).
- **Add to ID token**: on.
- **Add to access token**: on.
- **Add to userinfo**: on.

Save. Then **Groups, Create group**: `cooktrace-admins`. Add your admin user to it (**Users, &lt;user&gt;, Groups, Join group**).

## TraceApps side

Add to your `.env`:

```env
OIDC_ISSUER=https://keycloak.example.com/realms/home
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...paste from Credentials tab...
OIDC_REDIRECT_URIS=https://cook.example.com/api/oidc/callback
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

Restart:

```bash
docker compose up -d
```

## Verify

1. Load the app in a private window. A **Sign in with Keycloak** button appears next to the local login form.
2. Click. Keycloak redirects to its login page, you sign in, and you land back inside the app already logged in.
3. In **Settings, Users** the provider row is present with a padlock badge (env-locked).
4. If you set up the group mapper, an account in `cooktrace-admins` shows `role=admin` on its profile row.

## Troubleshooting

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
