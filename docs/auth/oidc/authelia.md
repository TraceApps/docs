# OIDC recipe: Authelia

[Authelia](https://www.authelia.com/) is a self-hosted SSO + 2FA portal with an OpenID Connect provider. If you already run it in front of other services, TraceApps drops in as another client under `identity_providers.oidc.clients`. This recipe assumes an Authelia instance with file-based configuration you can edit, and was checked against Authelia 4.39. The examples use CookTrace at `https://cook.example.com`.

!!! note "Authelia 4.39 and later"
    Since 4.39, Authelia leaves the user's email, name, username and groups out of the ID token by default and only serves them from its userinfo endpoint. TraceApps reads them from there when the ID token lacks them, so the claims policy in step 2 is optional; it puts them in the ID token as well.

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

### 2. Define the claims policy and the client

Generate the client secret first. This prints a random password (for the app's `OIDC_CLIENT_SECRET`) and its digest (for Authelia's `client_secret`):

```bash
authelia crypto hash generate pbkdf2 --variant sha512 --random --random.length 72 --random.charset rfc3986
```

Then add a claims policy that puts what TraceApps needs into the ID token, and the client that uses it. Both go into the same `identity_providers.oidc` block as step 1 (merge them in; don't add a second `identity_providers:` key):

```yaml
identity_providers:
  oidc:
    claims_policies:
      traceapps:
        id_token: ['email', 'email_verified', 'preferred_username', 'name', 'groups']
    clients:
      - client_id: 'cooktrace'
        client_name: 'CookTrace'
        client_secret: '$pbkdf2-sha512$310000$...the Digest from the command above...'
        public: false
        authorization_policy: 'one_factor'
        claims_policy: 'traceapps'
        require_pkce: true
        pkce_challenge_method: 'S256'
        redirect_uris:
          - 'https://cook.example.com/api/auth/oidc/callback/1'
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

Worth calling out:

- The `1` in the redirect URI is the provider ID in the app; see [The callback URL](../oidc.md#the-callback-url).
- Authelia's docs describe ID-token claims policies as an escape hatch for apps that don't read the userinfo endpoint, which is the case here.
- `client_secret` should be the digest. Authelia still accepts a plaintext secret, but that is deprecated.
- `authorization_policy: one_factor` lets password-authenticated users through. Use `two_factor` to require Authelia's 2FA on every sign-in into the app.
- By default Authelia asks for consent on each sign-in. To remember it, set `pre_configured_consent_duration` on the client (for example `'1 month'`).

### 3. Note the issuer URL

Authelia's issuer is the base URL of your Authelia instance, no path suffix:

```
https://auth.example.com
```

Confirm by opening `/.well-known/openid-configuration` under that host and checking that the `issuer` field matches. That value goes into `OIDC_ISSUER`.

### 4. (Optional) Group claim

Authelia groups (from `users_database.yml` or LDAP) arrive in the `groups` claim, as long as the client has the `groups` scope and the claims policy lists `groups` (both above).

## TraceApps side

Add to the app's environment, for example in `.env` next to the compose file (the [install examples](../../getting-started/compose.md) load it with `env_file: .env`):

```env
OIDC_ISSUER=https://auth.example.com
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...the Random Password from step 2...
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_DISPLAY_NAME=Authelia
OIDC_SCOPE=openid profile email groups
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

For group-based admin (recommended if you already have an admin group):

```env
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=admins
```

At each SSO sign-in, members of the `admins` group in Authelia become admins in the app and everyone else becomes a regular user, so make sure your own account is in it. Adjust the value to match whatever your group is called.

Recreate the container (a plain restart keeps the old values):

```bash
docker compose up -d
```

The log should show `[oidc-env] Loaded 1 OIDC provider from environment (IDs: 1)`. If the ID isn't `1`, use that number in the redirect URI, in both Authelia and `OIDC_REDIRECT_URIS`.

## Verify

1. Open the app in a private window. A **Sign In with Authelia** button appears next to the local login form.
2. Click it. Authelia asks for whatever `authorization_policy` requires (password only for `one_factor`, password + 2FA for `two_factor`), then redirects back.
3. You land inside the app, signed in.
4. In **Settings → Authentication**, the Authelia row appears with an **env** padlock badge.

## Signing out

Authelia doesn't support RP-initiated logout yet, so signing out of the app doesn't end the Authelia session. The next **Sign In with Authelia** passes straight through while that session lasts. Sign out at Authelia itself if you need to switch users.

## Troubleshooting

!!! warning "Blank page after signing in at Authelia"
    The redirect URI is wrong, so Authelia sent you back to a path the app doesn't handle. It must be exactly `https://<your-host>/api/auth/oidc/callback/<id>`, the same in Authelia and in `OIDC_REDIRECT_URIS`.

!!! warning "Sign-in fails at the token exchange (`invalid_client`)"
    `OIDC_CLIENT_SECRET` must be the random password the command printed, and Authelia's `client_secret` its digest. Regenerate both with the command in step 2 if in doubt.

!!! warning "No email, odd usernames, or no groups"
    Check that the client's `scopes:` and `OIDC_SCOPE` both include `profile`, `email` and, for groups, `groups`: Authelia only releases what the client may ask for. Sign out and back in after changing it.

!!! tip "Two-factor for admins only"
    To let everyday users in with one factor but require two for admins, define an OIDC authorization policy under `identity_providers.oidc.authorization_policies` (a `default_policy` plus a rule with `subject: 'group:admins'` and `policy: 'two_factor'`), and set the client's `authorization_policy` to its name. Authelia's regular `access_control` rules don't apply to OIDC sign-ins.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Authentik recipe](authentik.md)
