# OIDC recipe: Authentik

Authentik is the most common IdP self-hosters pair with TraceApps. This recipe assumes an Authentik instance you can log into as an administrator, and was checked against authentik 2026.8. Adjust hostnames to match your setup; the examples use CookTrace at `https://cook.example.com`.

## Authentik side

### 1. Create the application and provider

In Authentik's admin interface, go to **Applications → Applications → New Application**. The wizard creates the application and its provider together.

**Application:**

- **Name**: `CookTrace` (or LiftTrace, NoteTrace, NutriTrace, whichever app).
- **Slug**: `cooktrace`. The slug becomes part of the issuer URL.
- **Launch URL** (optional): `https://cook.example.com/`.

**Provider type:** **OAuth2/OpenID Provider**.

**Provider:**

- **Authorization flow**: `default-provider-authorization-implicit-consent` for a smooth "already signed in" experience, or the explicit-consent variant if you want a consent screen the first time.
- **Client type**: `Confidential`.
- **Client ID**: keep the generated value, or set something readable like `cooktrace`.
- **Client Secret**: keep the generated value. Copy it now; you need it in a moment.
- **Redirect URIs/Origins**: add a row with matching mode **Strict**, type **Authorization**, and the URL `https://cook.example.com/api/auth/oidc/callback/1`. The `1` is the provider ID in the app; see [The callback URL](../oidc.md#the-callback-url). Add one row per address if you run staging and prod.
- **Signing Key**: `authentik Self-signed Certificate` is fine.
- **Scopes** (under advanced protocol settings): keep the defaults (`openid`, `email`, `profile`). The default `profile` mapping already includes the user's groups in a `groups` claim, so there's nothing extra to add for group-based admin. Leave **Include claims in id_token** on (the default): the app reads the ID token.

**Bindings:** to limit who can sign in, bind a group to the application (the wizard's bindings step, or the application's **Policy / Group / User Bindings** tab). With no bindings, every Authentik user can sign in.

Finish the wizard.

### 2. Optional: sign-out redirect

When someone signs out of the app, the app also ends their Authentik session and asks Authentik to send them back. For that, add two more rows to **Redirect URIs/Origins** with type **Post Logout**: `https://cook.example.com/` and, if you use the Android app, `cooktrace://oidc-callback` (`lifttrace://`, `notetrace://` or `nutritrace://` for the other apps). Without them, sign-out still works; Authentik just shows its own signed-out page instead of returning to the app.

### 3. Optional: let existing accounts link by email

Since authentik 2025.10, the default `email` mapping sends `email_verified: false` for everyone, and the app only links an SSO sign-in to an existing account when the email is verified. So by default, when someone who already has an account in the app signs in through Authentik for the first time:

- with `OIDC_AUTO_REGISTER=0` (the default), the sign-in is refused, and
- with `OIDC_AUTO_REGISTER=1`, the app creates a **second, empty account** with the same email, and they land in that one instead of their own.

Either way, they can link Authentik to their real account themselves: sign in with their password, then add the provider under **Linked Accounts** in their profile. After that, SSO signs them straight into it.

If you trust the email addresses in your Authentik, you can have them treated as verified instead: under **Customization → Property Mappings**, create a **Scope Mapping** with scope name `email` and this expression, then select it in place of the default `email` mapping on the provider:

```python
return {
    "email": request.user.email,
    "email_verified": True,
}
```

### 4. Copy the issuer URL

On the provider's page, copy **OpenID Configuration Issuer** (something like `https://auth.example.com/application/o/cooktrace/`). That is your `OIDC_ISSUER`.

## TraceApps side

Add to the app's environment, for example in `.env` next to the compose file (the [install examples](../../getting-started/compose.md) load it with `env_file: .env`):

```env
OIDC_ISSUER=https://auth.example.com/application/o/cooktrace/
OIDC_CLIENT_ID=cooktrace
OIDC_CLIENT_SECRET=...paste the client secret...
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_DISPLAY_NAME=Authentik
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

If you want group-based admin, add:

```env
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=cooktrace-admins
```

Then in Authentik, create a group called `cooktrace-admins` and put your admin account in it. At each SSO sign-in, members become admins and everyone else becomes a regular user, so add yourself before turning this on.

If you'd like people without an account in the app to get one automatically at their first SSO sign-in, instead of needing an invite:

```env
OIDC_AUTO_REGISTER=1
```

Save the file and recreate the container (a plain restart keeps the old values):

```bash
docker compose up -d
```

The log should show `[oidc-env] Loaded 1 OIDC provider from environment (IDs: 1)`. If the ID isn't `1`, use that number in the callback URL, in both Authentik and `OIDC_REDIRECT_URIS`.

## Verify

1. Open the app in a private window. The login page shows a **Sign In with Authentik** button (or whatever you set `OIDC_DISPLAY_NAME` to).
2. Click it. Authentik takes over, asks for credentials, and redirects back.
3. You land inside the app, signed in.
4. In **Settings → Authentication**, the provider appears with an **env** padlock badge (defined through env vars, so it can be viewed but not edited there).

## Troubleshooting

!!! warning "Blank page after signing in at Authentik"
    The callback URL is wrong, so Authentik sent you back to a path the app doesn't handle. It must be exactly `https://<your-host>/api/auth/oidc/callback/<id>`, the same in Authentik and in `OIDC_REDIRECT_URIS`. Older versions of this page showed `/api/oidc/callback`, which doesn't work.

!!! warning "`redirect_uri` error at Authentik"
    Authentik refuses to continue if the URL in `OIDC_REDIRECT_URIS` doesn't match one of the provider's **Authorization** redirect rows. With **Strict** matching, trailing slashes, `http` vs `https`, and port numbers all count. Paste the same string into both places.

!!! warning "Existing users are refused, or land in a new empty account"
    Their email matches an account in the app, but Authentik marked it unverified, so the app won't link it automatically. Link it from the profile, or use the custom email mapping; step 3 above has both. If someone already ended up in a second account, an admin deletes that account in **Settings → Users** first (it holds the Authentik link), then they link Authentik from their real account.

!!! warning "Group-based admin doesn't apply"
    Check that the `profile` scope is still selected on the provider (it carries `groups`), that **Include claims in id_token** is on, and that `OIDC_ADMIN_GROUP_VALUE` is the group's exact name. The role is set at sign-in, so sign out and back in after changing it.

!!! tip "Test discovery with `curl` first"
    From the host running the app's container:

    ```bash
    curl -s https://auth.example.com/application/o/cooktrace/.well-known/openid-configuration | jq .issuer
    ```

    Your issuer echoed back means discovery works. A `404` means the slug in the URL is wrong.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Keycloak recipe](keycloak.md)
