# OIDC recipe: Pocket-ID

[Pocket-ID](https://github.com/pocket-id/pocket-id) is a lightweight, passkey-only OIDC provider that a lot of self-hosters run for household or homelab SSO. No passwords, no TOTP, just WebAuthn passkeys registered per user. It pairs cleanly with TraceApps because the wire contract is a plain OIDC discovery + PKCE flow, nothing exotic.

This recipe assumes a running Pocket-ID instance on a URL you can reach from the TraceApps container (for example `https://id.home.arpa`) and admin access to that Pocket-ID.

## Pocket-ID side

### 1. Create a Client

In the Pocket-ID admin UI, open **OIDC Clients, Add Client**.

- **Name**: `CookTrace` (or LiftTrace, NutriTrace, whichever app).
- **Callback URLs**: `https://cook.example.com/api/auth/oidc/callback`. Add one per app-instance you run (staging + prod, sibling subdomains, etc).
- **Logout Callback URLs**: `https://cook.example.com/` if you want post-logout to land back on the app.
- **PKCE**: leave enabled. TraceApps sends `code_challenge_method=S256` so this Just Works.
- **Public Client**: leave off. TraceApps stores the secret server-side, so a confidential client is the right choice.

Save. On the client's detail page grab three values:

- **Client ID** (an opaque string Pocket-ID generated).
- **Client Secret** (revealed once, so copy it now).
- **Issuer URL**, which is just your Pocket-ID base URL, no path suffix (`https://id.home.arpa`).

### 2. Allow the users who should be able to sign in

Pocket-ID gates each client by user group. Under the client's **Allowed User Groups** (or the equivalent in your version), add the group whose members should be able to SSO into TraceApps. If you don't restrict, every Pocket-ID user is allowed through.

### 3. Register passkeys

Because Pocket-ID is passkey-only, each user who intends to sign in via SSO needs a passkey on file **before** they hit the TraceApps login. If they don't, the Pocket-ID login page has nothing to authenticate them with and the SSO flow dead-ends. Have each user log into Pocket-ID directly once and register a passkey (fingerprint, Face ID, hardware key, browser passkey, whatever their device offers).

## TraceApps side

Add to your `.env` next to the compose file:

```env
OIDC_ISSUER=https://id.home.arpa
OIDC_CLIENT_ID=<paste Client ID>
OIDC_CLIENT_SECRET=<paste Client Secret>
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback
OIDC_DISPLAY_NAME=Pocket-ID
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

For a household setup where you want new Pocket-ID users to get a TraceApps account automatically on first sign-in:

```env
OIDC_AUTO_REGISTER=1
```

Restart the container:

```bash
docker compose up -d
```

## Verify

1. Open the app in a private window. The login page should show a **Sign in with Pocket-ID** button.
2. Click it. Pocket-ID prompts for a passkey. Authenticate.
3. You land back inside TraceApps, logged in.
4. In **Settings, Users, OIDC providers**, the Pocket-ID row appears with a padlock badge (env-locked, admin UI can view but not edit).

## Troubleshooting

!!! warning "No passkey enrolled"
    If a user has never logged into Pocket-ID directly, the passkey prompt has nothing to match and the SSO flow fails silently. Have the user visit the Pocket-ID URL, sign in with a magic link (or however your instance enrolls new users), and register a passkey first. Then retry TraceApps SSO.

!!! warning "`redirect_uri` mismatch"
    Pocket-ID rejects the callback if the URL in `OIDC_REDIRECT_URIS` does not exactly match one registered on the client. Trailing slashes, `http` vs `https`, and port numbers all matter. Paste the same string into both places.

!!! tip "Test discovery with `curl`"
    From the host running your TraceApps container:

    ```bash
    curl -s https://id.home.arpa/.well-known/openid-configuration | jq .issuer
    ```

    A `200` with your Pocket-ID URL echoed back means discovery works.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Authentik recipe](authentik.md)
