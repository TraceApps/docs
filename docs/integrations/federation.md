# Federation (cross-app links)

The three TraceApps siblings can talk to each other over HTTP. Some flows keep NutriTrace as the server (foods, workouts). Others put CookTrace on the server side so NutriTrace can pull recipes out of it, same shape as NT's Mealie integration. Each app knows how to be a client and how to be a server, depending on the flow.

This page is the high-level story. Each flow has its own configuration page linked at the bottom, and the exact wire contracts live at [Federation API (v1) on NutriTrace](../nutritrace/federation-api.md) and (soon) an equivalent page on the CookTrace side.

## What flows where

- **CookTrace pulls foods from NutriTrace.** When you build a recipe in CookTrace, ingredient rows can auto-populate nutrition from your NutriTrace foods library (barcode match preferred, name match as fallback). No more re-typing calories for the same tin of chickpeas you already logged in NT last week.
- **NutriTrace pulls recipes from CookTrace.** On the NT Foods search, Recipes tab, a **CookTrace** source chip appears next to Local and From Others once you have configured the connection. Pick a recipe, it opens in NT's Recipe editor with per-ingredient snapshots and rollup totals, ready to save into your NT recipes catalog and log from the diary. Mirrors the Mealie source chip on the Foods tab. See [Pull a CookTrace recipe into NutriTrace](../cooktrace/nt-federation.md#pull-a-cooktrace-recipe-into-nutritrace).
- **LiftTrace pushes workouts to NutriTrace.** When you finish a lift session in LT, it posts a workout summary (name, duration, kcal burned) to NT. That daily kcal-out then feeds NT's Dynamic and Adaptive calorie-goal modes, so your daily target reflects the fact that you actually lifted this morning.

Every flow is opt-in, per-user, and configured from the app that acts as the client (the one initiating the request).

## How auth works

Both NutriTrace and CookTrace expose a versioned federation API at `/api/v1/`. Every request needs a Bearer token in the `Authorization` header. Tokens are personal access tokens: minted inside whichever app hosts the data, scoped to what the holder is allowed to do, and hashed at rest so the raw value only ever exists once (at creation, shown to you to copy).

Token format: NT tokens start with `nt_pat_`, CT tokens with `ct_pat_`, followed by 43 base64url characters. The prefix makes leaked tokens easy to spot.

Scopes today (across both apps):

- `read:foods` (NT). Read a user's foods library. CookTrace needs this to pull foods into its pantry.
- `read:recipes` (CT). Read a user's recipes catalog. NutriTrace needs this to pull CT recipes into its Foods search.
- `write:workouts` (NT). Log workouts into a user's wellness history. LiftTrace needs this.
- `write:activity`, `write:body-measurements` (NT). External trackers and headless integrations.

Tokens are per-user (not per-instance): the token identifies which account the calls act on. If two family members share one instance and both want federation, they each mint their own token.

## Configuring it

Federation always starts on the app that owns the data by minting a token, then the token gets pasted into the app that initiates the request.

**On NutriTrace**: sign in as the account you want federated, open Settings, expand **API Tokens**, click **New Token**. Give it a name (something like "CookTrace pantry pull" or "LiftTrace on the desktop") and tick the scopes the client will need. On save the raw token is shown once, formatted `nt_pat_<43 chars>`. Copy it right then, because NT only stores the hash after that.

**On CookTrace**: sign in as the account whose recipes you want NT to pull, open Settings, expand **API Tokens**, click **New Token**. Give it a name like "NutriTrace recipe pull" and tick `read:recipes`. Same format (`ct_pat_<43 chars>`), same one-shot reveal.

**On the client side** (the app pulling or pushing): open Settings, find the section for the other app (**NutriTrace federation** on CookTrace/LiftTrace, **CookTrace** under Connected Services on NutriTrace), paste the instance URL and token, click Save. The client hits `/api/v1/me` on the target and echoes back the username on success, or a specific error if the URL is wrong, the token is invalid, or the scopes are missing.

## Transport, revocation, rotation

All traffic goes over whatever transport your NT instance is served on. HTTPS is the sensible default; plain HTTP works on a trusted LAN if that's what you're running. The client app forwards the request server-side (never browser to NT), so the token stays on the sibling app's server and doesn't reach the WebView.

Rate limit: NT enforces `API_RATE_LIMIT_PER_MIN` per-token (default 60 requests per minute per token). Bump the env var if you have a chatty use case.

Rotate a token by minting a new one, updating the client, then deleting the old one from NT's API Tokens list. Revocation is immediate: the next call from the old token returns `401 auth_invalid`. There is no grace period.

## Related

- [NutriTrace federation (CookTrace side)](../cooktrace/nt-federation.md)
- [NutriTrace federation (LiftTrace side)](../lifttrace/nt-federation.md)
- [Federation API (v1)](../nutritrace/federation-api.md)
