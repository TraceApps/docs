# OIDC recipe: Authelia

[Authelia](https://www.authelia.com/) is a self-hosted SSO + 2FA portal that speaks OIDC as of the 4.38 series. If you already run it in front of other services, TraceApps drops in as another client under `identity_providers.oidc.clients`. This recipe assumes an Authelia instance with OIDC enabled (or that you're about to enable it) and file-based configuration you can edit.

## Authelia side

### 1. Enable the OpenID Connect provider

In your `configuration.yml`, add or extend the `identity_providers.oidc` block. The `hmac_secret` and `jwks` keys are required and must be stable across restarts, otherwise every issued token is invalidated on reboot.

```yaml
identity_providers:
  oidc:
    hmac_secret: '<64+ random chars, e.g. openssl rand -hex 32>'
    jwks:
      - key_id: 'default'
        algorithm: 'RS256'
        use: 'sig'
        key: |
          -----BEGIN PRIVATE KEY-----
          ...contents of a 2048-bit RSA private key...
          -----END PRIVATE KEY-----
    clients: []   # filled in below
```

Generate an RSA key with `openssl genrsa -out oidc.key 2048` and paste the contents into `key:` (or use Authelia's `authelia crypto pair rsa generate` helper).

### 2. Define the client

Under `identity_providers.oidc.clients`, add a block for the app:

```yaml
    clients:
      - client_id: 'cooktrace'
        client_name: 'CookTrace'
        client_secret: '$pbkdf2-sha512$310000$...'
        public: false
        authorization_policy: 'one_factor'
        redirect_uris:
          - 'https://cook.example.com/api/auth/oidc/callback'
        scopes:
          - 'openid'
          - 'profile'
          - 'email'
          - 'groups'
        response_types:
          - 'code'
        grant_types:
          - 'authorization_code'
        token_endpoint_auth_method: 'client_secret_post'
```

Two things worth calling out:

- `client_secret` must be the **hashed** form, not the plaintext. Generate the pair with `authelia crypto hash generate pbkdf2 --variant sha512 --iterations 310000 --password '<your secret>'`. The command prints both the plaintext (keep it for the TraceApps env var) and the hash (paste into `client_secret:`).
- `authorization_policy: one_factor` lets password-authenticated users through. Switch to `two_factor` to require Authelia's 2FA (TOTP, WebAuthn, Duo) on every SSO into TraceApps. That gate applies here just like it does for any other Authelia-fronted app.

### 3. Note the issuer URL

Authelia's issuer is the base URL of your Authelia instance, no path suffix:

```
https://auth.example.com
```

Confirm by hitting `/.well-known/openid-configuration` under that host and checking the `issuer` field matches exactly. That value goes into `OIDC_ISSUER`.

### 4. (Optional) Group claim

If you use Authelia groups (from `users_database.yml` or LDAP) to gate admin access, they're emitted in the `groups` claim when you include the `groups` scope on the client (as above). No extra mapper needed.

## TraceApps side

Add to your `.env`:

```env
OIDC_ISSUER=https://auth.example.com
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...plaintext secret from step 2...
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback
OIDC_DISPLAY_NAME=Authelia
OIDC_SCOPE=openid profile email groups
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

For group-based admin promotion (recommended if you already have an admin group):

```env
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=admins
```

Members of the `admins` group in Authelia pick up `role='admin'` on the next SSO login. Adjust the value to match whatever your group is called.

Restart the container:

```bash
docker compose up -d
```

## Verify

1. Open the app in a private window. A **Sign in with Authelia** button appears next to the local login form.
2. Click it. Authelia takes over, prompts for whatever `authorization_policy` requires (password only for `one_factor`, password + 2FA for `two_factor`), then redirects back.
3. You land inside the app, logged in.
4. In **Settings, Users, OIDC providers**, the Authelia row appears with a padlock badge.

## Troubleshooting

!!! warning "Client secret hash mismatch"
    Pasting the plaintext secret into `client_secret:` in `configuration.yml` looks like it works until you try to sign in, at which point Authelia rejects the token exchange. Regenerate the hash with `authelia crypto hash generate pbkdf2 ...` and paste the `$pbkdf2-sha512$...` output.

!!! warning "Empty groups claim"
    If `groups` never resolves in the ID token, double-check that `scopes:` on the client includes `groups`, and that `OIDC_SCOPE` on the TraceApps side asks for it too. Re-sign-in from a fresh session so the token isn't cached.

!!! tip "Two-factor for admin only"
    If you want casual users on one-factor but admins forced through 2FA, keep `authorization_policy: one_factor` here and set up an Authelia access-control rule that requires two-factor for the specific admin group. TraceApps doesn't care which factor Authelia used; it just consumes the ID token.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Authentik recipe](authentik.md)
