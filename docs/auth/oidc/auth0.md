# OIDC recipe: Auth0

[Auth0](https://auth0.com/) is a hosted identity platform that speaks OIDC natively. A free tenant is enough for a small self-hosted deployment. This recipe assumes you have an Auth0 tenant (`your-tenant.us.auth0.com`, or whatever region prefix Auth0 assigned you) and admin access to its dashboard.

## Auth0 side

### 1. Create an Application

In the Auth0 dashboard, **Applications, Applications, Create Application**.

- **Name**: `CookTrace` (or LiftTrace, NutriTrace).
- **Application type**: `Regular Web Applications`. Do not pick SPA, that flow doesn't fit a server-side app that holds a client secret.

Create. On the new application's page, go to the **Settings** tab.

### 2. Fill in the callback settings

Scroll to **Application URIs** and set:

- **Allowed Callback URLs**: `https://cook.example.com/api/auth/oidc/callback`
- **Allowed Logout URLs**: `https://cook.example.com/`
- **Allowed Web Origins**: `https://cook.example.com`

Auth0 accepts a comma-separated list, so add staging and prod on separate lines if you run both.

Scroll to the top of **Settings** and copy:

- **Domain** (`your-tenant.us.auth0.com` or similar).
- **Client ID**.
- **Client Secret** (click the reveal icon).

Save Changes at the bottom.

### 3. (Optional) Emit a group / role claim

Auth0 does not put roles into the ID token by default. If you want to use `OIDC_ADMIN_GROUP_CLAIM` to promote users to TraceApps admin based on their Auth0 role, add a Post-Login Action.

**Actions, Library, Build Custom, Post Login trigger**. Paste:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  const roles = event.authorization?.roles || [];
  api.idToken.setCustomClaim('https://traceapps/roles', roles);
};
```

The claim name has to be a URI in Auth0's world (they reject unprefixed custom claim names on tokens issued to third-party apps). Deploy the action, then **Actions, Triggers, post-login**, drag your action into the flow, apply.

Then **User Management, Roles, Create Role**: `cooktrace-admin`. Assign it to the users who should be admins.

## TraceApps side

Add to your `.env`:

```env
OIDC_ISSUER=https://your-tenant.us.auth0.com/
OIDC_CLIENT_ID=<paste Client ID>
OIDC_CLIENT_SECRET=<paste Client Secret>
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback
OIDC_DISPLAY_NAME=Auth0
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

Note the trailing slash on `OIDC_ISSUER`, Auth0 includes it and the discovery endpoint expects it. If you drop it you'll see `issuer mismatch` errors on callback.

For role-based admin promotion (matches the Action from step 3):

```env
OIDC_ADMIN_GROUP_CLAIM=https://traceapps/roles
OIDC_ADMIN_GROUP_VALUE=cooktrace-admin
```

For self-service registration:

```env
OIDC_AUTO_REGISTER=1
```

Restart the container:

```bash
docker compose up -d
```

## Verify

1. Open the app in a private window. A **Sign in with Auth0** button appears next to the local login form.
2. Click it. Auth0's Universal Login page renders (or your custom Auth0 UI), you sign in, and Auth0 redirects back.
3. You land inside the app, logged in.
4. In **Settings, Users, OIDC providers**, the Auth0 row appears with a padlock badge (env-locked).

## Troubleshooting

!!! warning "`issuer mismatch` on callback"
    Auth0's issuer is `https://your-tenant.us.auth0.com/` with a trailing slash. If you set `OIDC_ISSUER` without the slash, discovery works but the callback rejects the ID token because its `iss` claim doesn't match. Add the slash.

!!! warning "Custom claim not visible in ID token"
    Auth0 silently drops custom claims that don't have a URI-shaped name. `roles` won't work; `https://traceapps/roles` will. The URI doesn't have to resolve to anything, it just has to look like a URI.

!!! tip "Free tier is plenty"
    Auth0's free tier covers 7,500 monthly active users, which is orders of magnitude more than a household or small-team self-hosted app needs. No credit card required.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Google Workspace recipe](google.md)
