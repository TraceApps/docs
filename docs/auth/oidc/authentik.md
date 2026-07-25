# OIDC recipe: Authentik

Authentik is the most common IdP self-hosters pair with TraceApps, and it's the one the OIDC layer was originally tested against. This recipe assumes an Authentik instance you can log into as an administrator. Adjust hostnames to match your setup.

## Authentik side

### 1. Create a Provider

In Authentik's admin UI, go to **Applications, Providers, Create**. Pick **OAuth2/OpenID Provider**.

Fill in:

- **Name**: `CookTrace` (or LiftTrace, NutriTrace, whichever app).
- **Authentication flow**: `default-authentication-flow` (or your custom sign-in flow).
- **Authorization flow**: `default-provider-authorization-implicit-consent` for a smooth "already logged in" experience, or the explicit-consent variant if you want a consent screen the first time.
- **Client type**: `Confidential`.
- **Client ID**: leave the auto-generated value, or set something readable like `cooktrace`.
- **Client Secret**: leave the auto-generated value. Copy it now, you'll need it in a moment.
- **Redirect URIs**: `https://cook.example.com/api/oidc/callback` (one per line if you have staging + prod).
- **Signing Key**: `authentik Self-signed Certificate` is fine.
- **Scopes**: keep the defaults (`openid`, `profile`, `email`). If you want groups to flow through, also select `authentik default OAuth Mapping: OpenID 'groups'`.

Save.

### 2. Create an Application

**Applications, Applications, Create**.

- **Name**: `CookTrace`.
- **Slug**: `cooktrace`.
- **Provider**: the provider you just made.
- **Launch URL**: `https://cook.example.com/`.

Save. Assign the application to the group(s) that should be allowed to sign in.

### 3. Grab the issuer URL

From the provider's detail page, copy the value of **OpenID Configuration Issuer** (something like `https://auth.example.com/application/o/cooktrace/`). This is what goes into `OIDC_ISSUER`. Note the trailing slash, Authentik includes it and the discovery request expects it.

## TraceApps side

Add to your `.env` next to the compose file:

```env
OIDC_ISSUER=https://auth.example.com/application/o/cooktrace/
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...paste the client secret...
OIDC_REDIRECT_URIS=https://cook.example.com/api/oidc/callback
OIDC_DISPLAY_NAME=Authentik
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

If you want group-based admin promotion (recommended), add:

```env
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=cooktrace-admins
```

Then in Authentik, create a group called `cooktrace-admins` and put your admin account in it. On the next SSO login the account picks up `role='admin'`.

If you'd like new IdP-side users to get an account automatically instead of needing an invite:

```env
OIDC_AUTO_REGISTER=1
```

Save the file, restart the container:

```bash
docker compose up -d
```

## Verify

1. Open the app in a private window. The login page should now show a **Sign in with Authentik** button (or whatever you set `OIDC_DISPLAY_NAME` to).
2. Click it. Authentik takes over, prompts for credentials, redirects back.
3. You land inside the app, logged in.
4. In **Settings, Users**, the Authentik provider appears with a padlock badge (env-locked, admin UI can view but not edit).

## Troubleshooting

!!! warning "`redirect_uri` mismatch"
    Authentik rejects the callback if the URL in `OIDC_REDIRECT_URIS` doesn't exactly match one of the URIs on the provider. Trailing slashes, `http` vs `https`, and port numbers all count. Paste the same string into both places.

!!! warning "Empty groups claim"
    If `ADMIN_GROUP_CLAIM=groups` never resolves, double-check that the provider's scope list includes the `groups` mapping (step 1 above). Without that mapping the ID token doesn't carry the claim.

!!! tip "Test discovery with `curl` first"
    You can sanity-check the issuer URL from the host running your container:

    ```bash
    curl -s https://auth.example.com/application/o/cooktrace/.well-known/openid-configuration | jq .issuer
    ```

    A `200` with your issuer echoed back means discovery works. A `404` means the trailing slash or the slug is wrong.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Keycloak recipe](keycloak.md)
