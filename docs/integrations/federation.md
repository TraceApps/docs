# Federation (cross-app links)

The three TraceApps siblings can talk to each other over HTTP. The point of the wiring: NutriTrace is the hub for anything nutrition-shaped (foods, workouts, calories in/out), and the other two apps read from it or write to it so you don't have to keep three parallel copies of the same data in your head.

This page is the high-level story. Each side has its own configuration page linked at the bottom, and the exact wire contract is at [Federation API (v1)](../nutritrace/federation-api.md).

## What flows where

- **CookTrace pulls foods from NutriTrace.** When you build a recipe in CookTrace, ingredient rows can auto-populate nutrition from your NutriTrace foods library (barcode match preferred, name match as fallback). No more re-typing calories for the same tin of chickpeas you already logged in NT last week.
- **LiftTrace pushes workouts to NutriTrace.** When you finish a lift session in LT, it posts a workout summary (name, duration, kcal burned) to NT. That daily kcal-out then feeds NT's Dynamic and Adaptive calorie-goal modes, so your daily target reflects the fact that you actually lifted this morning.

Both directions are opt-in, per-user, and configured from the client app's settings. NutriTrace is always the server side of the conversation; CookTrace and LiftTrace are always the clients.

## How auth works

NutriTrace exposes a versioned federation API at `/api/v1/`. Every request needs a Bearer token in the `Authorization` header. Tokens are personal access tokens: minted inside NT, scoped to what the holder is allowed to do, and hashed at rest so the raw value only ever exists once (at creation, shown to you to copy).

Two scopes matter today:

- `read:foods`. Required to read a user's foods library. CookTrace needs this.
- `write:workouts`. Required to log workouts into a user's wellness history. LiftTrace needs this.

Tokens are per-user (not per-instance): the token identifies which NT account the calls act on. If two family members share one NT instance and both want federation, they each mint their own token.

## Configuring it

Federation always starts on the NT side by minting a token, then the token gets pasted into the client app.

**On NutriTrace**: sign in as the account you want federated, open Settings, expand **API Tokens**, click **New Token**. Give it a name (something like "CookTrace" or "LiftTrace on the desktop") and tick the scopes the client will need. On save the raw token is shown once, formatted `nt_pat_<43 chars>`. Copy it right then, because NT only stores the hash after that.

**On CookTrace or LiftTrace**: open Settings, expand **NutriTrace federation**, paste your NT instance URL and the token, click **Save + Test**. The client hits NT's `/api/v1/me` and echoes back the username on success, or a specific error if the URL is wrong, the token is invalid, or the scopes are missing.

## Transport, revocation, rotation

All traffic goes over whatever transport your NT instance is served on. HTTPS is the sensible default; plain HTTP works on a trusted LAN if that's what you're running. The client app forwards the request server-side (never browser to NT), so the token stays on the sibling app's server and doesn't reach the WebView.

Rate limit: NT enforces `API_RATE_LIMIT_PER_MIN` per-token (default 60 requests per minute per token). Bump the env var if you have a chatty use case.

Rotate a token by minting a new one, updating the client, then deleting the old one from NT's API Tokens list. Revocation is immediate: the next call from the old token returns `401 auth_invalid`. There is no grace period.

## Related

- [NutriTrace federation (CookTrace side)](../cooktrace/nt-federation.md)
- [NutriTrace federation (LiftTrace side)](../lifttrace/nt-federation.md)
- [Federation API (v1)](../nutritrace/federation-api.md)
