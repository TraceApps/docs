# OIDC recipe: Pocket ID

[Pocket ID](https://github.com/pocket-id/pocket-id) is a lightweight, passwordless OIDC provider that a lot of self-hosters run for household or homelab SSO. Users sign in with passkeys (or, depending on your settings, a code sent by email or another device). It pairs cleanly with TraceApps because the wire contract is a plain OIDC discovery + PKCE flow, nothing exotic.

This recipe assumes a running Pocket ID instance on a URL you can reach from the app's container (for example `https://id.home.arpa`) and admin access to it. It was checked against Pocket ID 2.17. The examples use CookTrace at `https://cook.example.com`.

## Pocket ID side

### 1. Create the client

In Pocket ID's admin settings, open **OIDC Clients** and add a client:

- **Name**: `CookTrace` (or LiftTrace, NoteTrace, NutriTrace, whichever app).
- **Client type**: **Confidential Client**. TraceApps keeps the secret server-side.
- **Callback URLs**: `https://cook.example.com/api/auth/oidc/callback/1`. The `1` is the provider ID in the app; see [The callback URL](../oidc.md#the-callback-url). Add one per instance you run (staging + prod).

Save, and copy the **Client Secret** now: it's shown only once. The **Client ID** is generated (a UUID) unless you set your own.

### 2. Finish the client's settings

On the client's page:

- **PKCE**: turn it on. The app always sends a PKCE challenge (`S256`), and Pocket ID leaves PKCE off by default for confidential clients.
- **Logout Callback URLs**: `https://cook.example.com/`, plus `cooktrace://oidc-callback` if you use the Android app (`lifttrace://`, `notetrace://` or `nutritrace://` for the other apps). Signing out of the app also ends the Pocket ID session and returns here.
- **Skip Consent Screen** (optional): turn on if you don't want users asked for consent at their first sign-in.

### 3. Allow the users who should sign in

A new client is restricted to its **Allowed User Groups**, and starts with none, so **nobody can sign in until you add a group** (or remove the restriction). Add the group whose members should be able to sign in to the app.

### 4. Verified emails

Pocket ID sends `email_verified` from each user's verified flag, which is off unless the user verified their email or your instance has **Emails verified by default** turned on. The app only links an SSO sign-in to an existing account in the app when the email is verified. For people who already have an account, either verify their email in Pocket ID, or have them link Pocket ID from their profile (**Linked Accounts**) after signing in with their password. Without either, their first SSO sign-in is refused, or with `OIDC_AUTO_REGISTER=1` gets a second, empty account.

### 5. Make sure users can sign in to Pocket ID

Each user needs a way to sign in to Pocket ID itself (usually a registered passkey) before they try the app's SSO button. Have each user sign in to Pocket ID directly once and set it up.

## TraceApps side

Add to the app's environment, for example in `.env` next to the compose file (the [install examples](../../getting-started/compose.md) load it with `env_file: .env`):

```env
OIDC_ISSUER=https://id.home.arpa
OIDC_CLIENT_ID=<paste Client ID>
OIDC_CLIENT_SECRET=<paste Client Secret>
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_DISPLAY_NAME=Pocket ID
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

The issuer is your Pocket ID base URL, with no path.

For group-based admin, request the `groups` scope too (Pocket ID only sends the `groups` claim when it's asked for), and name the group:

```env
OIDC_SCOPE=openid profile email groups
OIDC_ADMIN_GROUP_CLAIM=groups
OIDC_ADMIN_GROUP_VALUE=cooktrace-admins
```

The value is the group's **Name** in Pocket ID (not its friendly name). At each SSO sign-in, members become admins and everyone else becomes a regular user, so add yourself to the group first.

For a household setup where new Pocket ID users should get an account in the app automatically on first sign-in:

```env
OIDC_AUTO_REGISTER=1
```

Recreate the container (a plain restart keeps the old values):

```bash
docker compose up -d
```

The log should show `[oidc-env] Loaded 1 OIDC provider from environment (IDs: 1)`. If the ID isn't `1`, use that number in the callback URL, in both Pocket ID and `OIDC_REDIRECT_URIS`.

## Verify

1. Open the app in a private window. The login page shows a **Sign In with Pocket ID** button.
2. Click it. Pocket ID asks you to sign in. Authenticate.
3. You land back inside the app, signed in.
4. In **Settings → Authentication**, the Pocket ID row appears with an **env** padlock badge (defined through env vars, so it can be viewed but not edited there).

## Troubleshooting

!!! warning "Pocket ID says you're not allowed to sign in"
    The user isn't in one of the client's **Allowed User Groups** (step 3). New clients allow nobody until a group is added.

!!! warning "Blank page after signing in at Pocket ID"
    The callback URL is wrong, so Pocket ID sent you back to a path the app doesn't handle. It must be exactly `https://<your-host>/api/auth/oidc/callback/<id>`, the same in Pocket ID and in `OIDC_REDIRECT_URIS`.

!!! warning "`redirect_uri` mismatch"
    Pocket ID rejects the request if the URL in `OIDC_REDIRECT_URIS` doesn't match one of the client's **Callback URLs**. `http` vs `https`, ports and trailing slashes all count. Paste the same string into both places.

!!! tip "Test discovery with `curl`"
    From the host running the app's container:

    ```bash
    curl -s https://id.home.arpa/.well-known/openid-configuration | jq .issuer
    ```

    Your Pocket ID URL echoed back means discovery works.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Authentik recipe](authentik.md)
