# OIDC recipe: Google Workspace

Google runs a fully compliant OIDC provider at `accounts.google.com` that any Google account (personal Gmail or Workspace) can authenticate against. For a self-hosted TraceApps instance you'd typically wire this up in one of two shapes:

- **Household**: any Google account can sign in, and you rely on TraceApps invites (or `OIDC_AUTO_REGISTER=0`) to gate who actually gets an account.
- **Workspace-only**: restrict to accounts in a specific Google Workspace domain, so only `@example.com` employees can complete the sign-in.

This recipe covers both. You'll need access to [Google Cloud Console](https://console.cloud.google.com/) with permission to create OAuth credentials in some project.

## Google Cloud side

### 1. Create (or pick) a project

In Cloud Console, use the project selector at the top and either pick an existing project you own or **New Project**, name it something like `TraceApps SSO`. Anything scoped inside this project (OAuth consent, client IDs) is what TraceApps will use.

### 2. Configure the OAuth consent screen

**APIs & Services, OAuth consent screen**.

- **User type**: `Internal` if your project is inside a Workspace and you want to restrict to that Workspace domain. Otherwise `External`.
- **App name**: `CookTrace` (or LiftTrace, NutriTrace).
- **User support email** and **Developer contact information**: your own address.
- **App domain, Application home page**: `https://cook.example.com`.
- **Authorized domains**: `example.com` (the domain you'll callback to).
- **Scopes**: add `openid`, `email`, `profile`. Nothing else needed.

Save. If you picked `External`, you'll be in Testing mode and only accounts you list under **Test users** can sign in until you publish the app. For a personal/household setup that's often fine; leave it in Testing and add the family members' Gmails as test users. Otherwise click **Publish app** to make it available to any Google account.

### 3. Create an OAuth 2.0 Client ID

**APIs & Services, Credentials, Create Credentials, OAuth client ID**.

- **Application type**: `Web application`.
- **Name**: `CookTrace`.
- **Authorized JavaScript origins**: `https://cook.example.com`.
- **Authorized redirect URIs**: `https://cook.example.com/api/auth/oidc/callback`.

Create. A dialog shows the **Client ID** (`...apps.googleusercontent.com`) and **Client secret**. Copy both, the secret can be re-fetched later but the ID is what goes into `OIDC_CLIENT_ID`.

## TraceApps side

Add to your `.env`:

```env
OIDC_ISSUER=https://accounts.google.com
OIDC_CLIENT_ID=<your-id>.apps.googleusercontent.com
OIDC_CLIENT_SECRET=...paste from Cloud Console...
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback
OIDC_DISPLAY_NAME=Google
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

Google's issuer URL has no trailing slash, no realm, no path. It's just `https://accounts.google.com`.

For a household setup where you'd like new Google users to get a TraceApps account automatically on first sign-in, add:

```env
OIDC_AUTO_REGISTER=1
```

Restart the container:

```bash
docker compose up -d
```

### Restrict to a Workspace domain (`hd`)

If you want to reject any Google account whose Workspace domain isn't yours (personal Gmail, another company's Workspace, etc), you have two overlapping options:

- **Consent-screen level**: set the OAuth consent screen to `Internal` (step 2). Google enforces this before the user even reaches the token exchange; anyone outside the Workspace sees an error and never gets an ID token.
- **Application level**: rely on the `hd` (hosted-domain) claim Google adds to the ID token for Workspace accounts. TraceApps doesn't natively filter by `hd`, but combined with `OIDC_AUTO_REGISTER=0` you get the equivalent effect: only pre-invited accounts (whose email you already vetted) can complete a sign-in.

For a strict "only my Workspace" gate the Internal consent screen is the cleaner story.

## Verify

1. Open the app in a private window. A **Sign in with Google** button appears next to the local login form.
2. Click it. Google's account picker takes over, you pick or sign in, and Google redirects back.
3. You land inside the app, logged in.
4. In **Settings, Users, OIDC providers**, the Google row appears with a padlock badge (env-locked).

## Troubleshooting

!!! warning "`redirect_uri_mismatch`"
    Google is strict: the URL in `OIDC_REDIRECT_URIS` must exactly match one of the **Authorized redirect URIs** on the client (scheme, host, port, path). Copy the same string into both places. No trailing slash on one side and not the other.

!!! warning "`access_denied` on published apps in Testing mode"
    If your consent screen is `External` and unpublished, only accounts on the **Test users** list can sign in. Add the user or click **Publish app** and go through Google's verification if you want the app open to any Google account.

!!! tip "Groups are not free on Google"
    Google's ID token does not carry Workspace group membership by default. If you want group-based admin promotion you'd need the Admin SDK / Directory API which is beyond the scope of a self-hosted app. For a small deploy, invite admins manually or promote them from Settings, Users after their first login.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Auth0 recipe](auth0.md)
