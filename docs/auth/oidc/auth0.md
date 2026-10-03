# OIDC recipe: Auth0

[Auth0](https://auth0.com/) is a hosted identity platform that speaks OIDC natively. A free tenant is enough for a small self-hosted deployment. The examples use CookTrace at `https://cook.example.com`. This recipe assumes you have an Auth0 tenant (`your-tenant.us.auth0.com`, or whatever region prefix Auth0 assigned you) and admin access to its dashboard.

## Auth0 side

### 1. Create an Application

In the Auth0 dashboard, **Applications → Applications → Create Application**.

- **Name**: `CookTrace` (or LiftTrace, NoteTrace, NutriTrace).
- **Application type**: `Regular Web Applications`. Not SPA: the app is a server that holds a client secret.

Create. On the new application's page, go to the **Settings** tab.

### 2. Fill in the callback settings

Scroll to **Application URIs** and set:

- **Allowed Callback URLs**: `https://cook.example.com/api/auth/oidc/callback/1`. The `1` is the provider ID in the app; see [The callback URL](../oidc.md#the-callback-url).
- **Allowed Logout URLs**: `https://cook.example.com/`, plus `cooktrace://oidc-callback` if you use the Android app (`lifttrace://`, `notetrace://` or `nutritrace://` for the other apps). Signing out of the app also ends the Auth0 session and returns here.

Both fields take a comma-separated list, so separate staging and prod with a comma if you run both.

Scroll to the top of **Settings** and copy:

- **Domain** (`your-tenant.us.auth0.com` or similar).
- **Client ID**.
- **Client Secret** (click the reveal icon).

Save Changes at the bottom. On the **Credentials** tab, set **Authentication Method** to **Client Secret (Post)**, to match `OIDC_TOKEN_AUTH_METHOD=client_secret_post` below.

If your tenant was created before November 2023, also turn on **RP-Initiated Logout End Session Endpoint Discovery** under the tenant's **Settings → Advanced**; without it, signing out of the app doesn't end the Auth0 session.

### 3. (Optional) Emit a role claim

Auth0 does not put roles into the ID token by default. If you want to use `OIDC_ADMIN_GROUP_CLAIM` to make users admins based on their Auth0 role, add a Post Login Action.

**Actions → Library → Create Action → Build from scratch**, trigger **Login / Post Login**. Paste:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  const roles = event.authorization?.roles || [];
  api.idToken.setCustomClaim('https://cook.example.com/roles', roles);
};
```

Use a URL you control as the claim name (it doesn't have to resolve, and must not be an Auth0 domain). A bare `roles` won't work: Auth0 reserves that name and silently drops the claim. **Deploy** the action, then go to **Actions → Flows → Login**, drag your action into the flow, and **Apply**.

Then **User Management → Roles → Create Role**: `cooktrace-admin`. Assign it to the users who should be admins.

## TraceApps side

Add to the app's environment, for example in `.env` next to the compose file (the [install examples](../../getting-started/compose.md) load it with `env_file: .env`):

```env
OIDC_ISSUER=https://your-tenant.us.auth0.com/
OIDC_CLIENT_ID=<paste Client ID>
OIDC_CLIENT_SECRET=<paste Client Secret>
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_DISPLAY_NAME=Auth0
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

Auth0's issuer is `https://<your domain>/`, with a trailing slash. The app works with or without it, as long as the domain is right.

For role-based admin promotion (matches the Action from step 3):

```env
OIDC_ADMIN_GROUP_CLAIM=https://cook.example.com/roles
OIDC_ADMIN_GROUP_VALUE=cooktrace-admin
```

For self-service registration:

```env
OIDC_AUTO_REGISTER=1
```

Recreate the container (a plain restart keeps the old values):

```bash
docker compose up -d
```

The log should show `[oidc-env] Loaded 1 OIDC provider from environment (IDs: 1)`. If the ID isn't `1`, use that number in the callback URL, in both Auth0 and `OIDC_REDIRECT_URIS`.

## Verify

1. Open the app in a private window. A **Sign In with Auth0** button appears next to the local login form.
2. Click it. Auth0's Universal Login page renders (or your custom Auth0 UI), you sign in, and Auth0 redirects back.
3. You land inside the app, signed in.
4. In **Settings → Authentication**, the Auth0 row appears with an **env** padlock badge.

## Troubleshooting

!!! warning "Blank page after signing in at Auth0"
    The callback URL is wrong, so Auth0 sent you back to a path the app doesn't handle. It must be exactly `https://<your-host>/api/auth/oidc/callback/<id>`, the same in Auth0 and in `OIDC_REDIRECT_URIS`.

!!! warning "Callback URL mismatch at Auth0"
    The URL in `OIDC_REDIRECT_URIS` must exactly match one of the **Allowed Callback URLs**. Paste the same string into both places.

!!! warning "Role claim missing from the ID token"
    Check that the Action is deployed and attached to the post-login trigger, and that the claim name is a URL (step 3); Auth0 silently drops reserved names such as `roles` and `groups`. `OIDC_ADMIN_GROUP_CLAIM` must be the exact same URL.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Google recipe](google.md)
