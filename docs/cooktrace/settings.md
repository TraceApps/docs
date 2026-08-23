# Settings reference

Every section on the Settings page, what it does, and where to find the deeper docs. Per-user sections apply only to the currently signed-in account. Admin sections apply instance-wide and only appear when the current user has the `admin` role.

Settings is laid out as collapsible cards grouped into **Display**, **Data & Tracking**, **Integrations**, **App**, **Admin**, and **About**. Use the search box at the top to jump straight to a section by name or by any keyword in its `SECTION_KEYWORDS` index.

## Profile (per-user)

Top of the Settings page. Avatar, name, role pill. Tap the card to open **/profile** for account edits: change display name, change password, sign out.

## Appearance (per-user)

- **Theme**: System, Dark, or Light.
- **Accent Color**: 12 preset chips, or a custom colour via the HSL/RGB/Hex sheet.
- **Navigation Style**: Bottom Tab Bar, Side Panel, or Both.
- **Persistent Sidebar**: keeps the sidebar mounted on viewports at or above 768px when navigation is set to Sidebar or Both.
- **Start Page**: which page CookTrace opens to (Recipes / Pantry / Diary / Shopping / Settings).
- **Reduce Motion**: dampens transitions and animations.
- **Page Banners**: Off, Gradient, or Animated.
- **Animation Style** (when Page Banners is Animated): Shimmer, Drift, Pulse, or Aurora.

## Regional and Units (per-user)

- **Language**: registered locales (English, Swedish today).
- **Date Format**: ISO, US, EU, or natural.
- **Time Format**: 12h or 24h.
- **Measurement System**: Imperial or Metric.
- **Energy**: kcal or kJ (independent of measurement system).

## Cooking (per-user)

- **Default Servings**: 1-20, used when adding a fresh recipe.
- **Auto-Add Ingredients to Pantry**: off by default. When on, saving a recipe creates any missing pantry rows for its ingredients.
- **Show Shared Recipes in My Main List**: off by default. When on, recipes shared with you blend into the primary Recipes tab instead of only appearing on Shared.
- **URL Import Engine**: Standard, Enhanced (server required), or Smart (Trace AI required). See [Import](import.md).
- **Enhanced Fallback** (visible when Enhanced is selected): Standard or Smart.
- **Shopping List > Default Grouping**: By Aisle, By Recipe, or Flat.
- **Shopping List > Checked Items**: Sink to Bottom or Hide.

## Nutrition (per-user)

- **Visible Nutriments**: pick which of the 31 nutriments in the catalog show inline on the Nutrition Facts box and the Pantry details sheet. Everything else still counts toward totals; this is a display filter.

## AI Assistant (per-user unless env-locked)

- **Enable Trace AI**.
- **Provider**: Anthropic Claude, OpenAI, Google Gemini, or OpenAI Compatible.
- **Model**: preset dropdown per provider, plus **Custom...** for free-text model IDs.
- **API Key**: saved server-side per user. **Save** triggers a connection test.
- **Base URL** (Custom or OpenAI Compatible only): e.g. `http://ollama:11434`.
- **Assistant Name**: display name for the chat panel (defaults to Trace).
- **Smart Log**: enables the hold-to-record voice input on the chat FAB.
- **Chef Hat**: toggles the chef-hat variant of the TraceFace mascot.
- **Env lock note**: when the operator sets `AI_*` env vars, fields lock and a note surfaces.

Deep dive: [Trace setup](../trace/setup.md).

## NutriTrace federation (per-user)

- **NT URL** and **Access Token**. Save and Test returns "Connected as <username>".
- **Pull foods / Import foods**: bulk-picker to backfill the CookTrace Pantry from your NutriTrace foods.
- **Log-a-cook fanout**: the client wiring for pushing cooked meals into NT's diary is on the roadmap.

Deep dive: [NutriTrace federation](../integrations/federation.md).

## Connected Services (per-user)

Food-lookup sources:

- **Open Food Facts** (OFF): Enable, Search Language, Search Country, Upload Country, and (only for OFF uploads) Account Username and Password.
- **USDA FoodData Central**: Enable and API Key.
- **Barcode Scanner**: Beep on Scan, Auto-Enable Flashlight (web only).

## Server Connection (Android only, per-user)

Only present in the Capacitor build. Set or change the server URL, disconnect, or switch between local (offline) and connected modes. See [Mobile modes](../mobile/modes.md).

## Notifications (per-user)

- **Device notifications** master toggle.
- **Push service**: Apprise, Gotify, or ntfy, one active at a time. Includes a **Send Test** button.
- **Apprise URL** and **Apprise Tag**.
- **Gotify URL** and **Gotify Token**.
- **Ntfy URL**, **Topic**, and optional **Token**.
- Per-reminder toggles: Cook Day Reminder, Thaw Alert, Shopping Nudge, Recipe Comments (default on), Weekly Summary.
- **Expiration Digest**: Enable, Days Before (1/3/5/7/14), Delivery Time (`HH:MM` container timezone).

Deep dives: [Apprise](../integrations/apprise.md), [Gotify](../integrations/gotify.md), [ntfy](../integrations/ntfy.md).

## Backup and Restore

- **Full Backup** (admin only, server mode): Create Backup, Upload & Restore, table of existing backups with Download / Restore / Delete.
- **Scheduled Backup** (env-lockable): Cadence Off/Daily/Weekly/Monthly, Time of Day, Retention Count.
- **Portable JSON Export**: Export JSON (no images), Import JSON (merges).
- **Local Full Backup** (Android local mode only): self-contained zip with base64-inlined images for phone-to-phone transfer.
- **Local Backup Schedule** (Android): device-side, persisted in `localStorage`.
- **Danger Zone**: Clear all data, Clear all settings.

Deep dive: [Backups](../self-hosting/backups.md), [Scheduled backups](../self-hosting/scheduled-backups.md).

## Import from Another App

The bulk import card. Drop a Mealie / Tandoor / Paprika archive or a CookTrace export. See [Import](import.md), [Migrate from Mealie](migrate-mealie.md).

## Kitchens (per-user)

- **Create Kitchen** with a name.
- Members panel per kitchen: invite by username, remove, per-user **Auto-Share Your Recipes** toggle.

Deep dive: [Kitchens](kitchens.md).

## Diagnostics (per-user)

- **Diagnostic Mode**: doubles the log ring buffer (1000 lines) and captures verbose output.
- **View Diagnostic Logs**: Copy, Share, Clear.
- **Share Log File**: Android only, when Diagnostic Mode is on. Shares the on-disk 7-day rotation.
- **Crash Report**: banner appears when the last session crashed. Share or Dismiss.

## Users (admin)

- **Enable User Management**: bootstraps an admin account on first turn-on.
- Users list with role, delete, reset password.
- **Invite user** by role and email. Requires SMTP; falls back to a copyable link when SMTP is unset.
- Pending invites table with revoke.

Deep dive: [Local users](../auth/local-users.md).

## Authentication (admin)

- **OIDC providers** table with Add via preset (Auth0, Authelia, Authentik, Google, Keycloak, Pocket-ID, Custom).
- Per-provider: issuer, `client_id`, `client_secret` (encrypted at rest), `redirect_uris`, `scope`, `token_endpoint_auth_method`, `admin_group_claim`, `admin_group_value`, `auto_link_verified_email`, `auto_register_new_users`, `is_active`, display name, logo URL.
- **Test discovery** button per provider.
- **Allow Password Login**: toggles password login on or off. Can be env-locked via `OIDC_ENABLE_EMAIL_PASSWORD_LOGIN`.

Deep dive: [OIDC](../auth/oidc.md), [SSO-only mode](../auth/sso-only.md).

## Email (admin)

- SMTP Host, Port, Secure (STARTTLS or SSL), User, Pass, From.
- **Save** and **Send Test Email** to a typed recipient.
- All fields env-lockable via `SMTP_*` env vars; locked fields disable in the UI.

Deep dive: [SMTP](../integrations/smtp.md).

## About

Version, platform tag (PWA or Android), Ko-fi link, description, disclaimer, license (AGPL-3.0).

## Related

- [Env vars](env-vars.md)
- [Trace AI setup](../trace/setup.md)
- [OIDC](../auth/oidc.md)
- [Backups](../self-hosting/backups.md)
