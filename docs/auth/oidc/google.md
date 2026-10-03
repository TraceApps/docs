# OIDC recipe: Google

Google runs an OIDC provider at `accounts.google.com` that any Google account (personal Gmail or Workspace) can authenticate against. For a self-hosted TraceApps instance you'd typically wire this up in one of two shapes:

- **Household**: any Google account can get through Google, and the app's own accounts decide who gets in (keep `OIDC_AUTO_REGISTER=0` and give people accounts first).
- **Organization-only**: only accounts in your Google Cloud organization (your Workspace) can sign in at all.

This recipe covers both. You'll need access to the [Google Cloud console](https://console.cloud.google.com/) with permission to create OAuth clients in some project. The examples use CookTrace at `https://cook.example.com`.

!!! warning "Google needs a public HTTPS address"
    Google only accepts redirect URIs on `https://`, on a domain name (not an IP address) whose ending is a public suffix such as `.com` or `.net`. A LAN-only name like `cook.lan` or `cook.home.arpa` is refused. If the app is only reachable on your LAN, use a self-hosted IdP instead.

## Google Cloud side

### 1. Create (or pick) a project

In the Cloud console, use the project selector at the top and either pick an existing project or create a new one, named something like `TraceApps SSO`.

### 2. Set up the Google Auth Platform

Open **Google Auth Platform** (search for it, or go through **APIs & Services → OAuth consent screen**). If the project has never used it, click **Get started** and fill in the basics. The settings live on these pages:

- **Branding**: **App name** `CookTrace` (or LiftTrace, NoteTrace, NutriTrace), **User support email**, **Developer contact information**, the **Application home page** `https://cook.example.com`, and under **Authorized domains** your domain (`example.com`).
- **Audience**: **User type** `Internal` if the project belongs to your Google Cloud organization and only its members should sign in; otherwise `External`.
- **Data Access**: the app only needs `openid`, `email` and `profile`.

Because the app asks only for `openid`, `email` and `profile`, an `External` app works for any Google account even while its publishing status is **Testing**: Google doesn't require a test-user list, show an unverified-app warning, or expire sign-ins for those scopes.

### 3. Create the client

On the **Clients** page, **Create client**:

- **Application type**: `Web application`.
- **Name**: `CookTrace`.
- **Authorized redirect URIs**: `https://cook.example.com/api/auth/oidc/callback/1`. The `1` is the provider ID in the app; see [The callback URL](../oidc.md#the-callback-url). Authorized JavaScript origins aren't needed.

Create. Copy the **Client ID** (`...apps.googleusercontent.com`) and the **Client secret** right away: Google shows the secret only at this point. New or changed redirect URIs can take from a few minutes to a few hours to start working.

## TraceApps side

Add to the app's environment, for example in `.env` next to the compose file (the [install examples](../../getting-started/compose.md) load it with `env_file: .env`):

```env
OIDC_ISSUER=https://accounts.google.com
OIDC_CLIENT_ID=<your-id>.apps.googleusercontent.com
OIDC_CLIENT_SECRET=...paste from Cloud Console...
OIDC_REDIRECT_URIS=https://cook.example.com/api/auth/oidc/callback/1
OIDC_DISPLAY_NAME=Google
OIDC_SCOPE=openid profile email
OIDC_TOKEN_AUTH_METHOD=client_secret_post
```

Google's issuer URL has no trailing slash, no realm, no path. It's just `https://accounts.google.com`.

For a household setup where you'd like new Google users to get a TraceApps account automatically on first sign-in, add:

```env
OIDC_AUTO_REGISTER=1
```

Recreate the container (a plain restart keeps the old values):

```bash
docker compose up -d
```

The log should show `[oidc-env] Loaded 1 OIDC provider from environment (IDs: 1)`. If the ID isn't `1`, use that number in the callback URL, in both Google and `OIDC_REDIRECT_URIS`.

### Restrict to your organization

If you want to reject any Google account outside your organization (personal Gmail, another company's Workspace, and so on), you have two overlapping options:

- **Google level**: set the **User type** to `Internal` (step 2). Google enforces this before the user reaches the app; anyone outside your organization gets an `org_internal` error and never gets an ID token.
- **App level**: keep `OIDC_AUTO_REGISTER=0` (the default). Then only people who already have an account in the app with the same email can sign in through Google; everyone else is refused. The app doesn't filter on Google's `hd` (hosted domain) claim itself.

For a strict "only my organization" gate, `Internal` is the cleaner option.

## Verify

1. Open the app in a private window. A **Sign In with Google** button appears next to the local login form.
2. Click it. Google's account picker takes over, you pick or sign in, and Google redirects back.
3. You land inside the app, signed in.
4. In **Settings → Authentication**, the Google row appears with an **env** padlock badge.

Signing out of the app doesn't sign you out of Google (Google doesn't offer that to apps), so the next **Sign In with Google** may pass straight through.

## Troubleshooting

!!! warning "`redirect_uri_mismatch`"
    Google is strict: the URL in `OIDC_REDIRECT_URIS` must exactly match one of the **Authorized redirect URIs** on the client (scheme, host, port, path). Copy the same string into both places. No trailing slash on one side and not the other.

!!! warning "Blank page after signing in at Google"
    The redirect URI is wrong, so Google sent you back to a path the app doesn't handle. It must be exactly `https://<your-host>/api/auth/oidc/callback/<id>`, the same in Google and in `OIDC_REDIRECT_URIS`.

!!! tip "Groups are not free on Google"
    Google's ID token does not carry Workspace group membership by default. If you want group-based admin promotion you'd need the Admin SDK / Directory API which is beyond the scope of a self-hosted app. For a small deploy, make people admins in **Settings → Users** instead.

## Related

- [OIDC / SSO overview](../oidc.md)
- [SSO-only mode and recovery](../sso-only.md)
- [Auth0 recipe](auth0.md)
