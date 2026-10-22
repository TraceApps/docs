# Settings reference

Every section on the Settings page, what it does, and where to find the deeper docs. Per-user sections apply only to the currently signed-in account. Admin sections apply instance-wide and only appear when the current user has the `admin` role.

Settings is laid out as collapsible cards grouped into **Display**, **Data & Tracking**, **Integrations**, **App**, **Admin**, and **About**. Use the search box at the top to jump straight to a section by name or by any keyword in its `SECTION_KEYWORDS` index.

## Profile (per-user)

Top of the Settings page. Avatar, name, role pill. Tap the card to open **/profile** for account edits: change display name, change password, sign out.

## Appearance (per-user)

- **Theme**: System, Dark, or Light.
- **Accent Color**: 12 preset chips, or a custom color via the HSL/RGB/Hex sheet.
- **Navigation Style**: Bottom Tab Bar, Side Panel, or Both.
- **Persistent Sidebar**: keeps the sidebar mounted on viewports at or above 768px when navigation is set to Sidebar or Both.
- **Start Page**: which page CookTrace opens to (Recipes / Pantry / Diary / Shopping / Settings).
- **Reduce Motion**: dampens transitions and animations.
- **Page Banners**: Off, Gradient, or Animated.
- **Animation Style** (when Page Banners is Animated): Shimmer, Drift, Pulse, or Aurora.

## Regional and Units (per-user)

- **Language**: the app's language: English, Español, Français or Svenska. Spanish covers every screen (from the next release), French about 40% so far; anything not yet translated shows in English. To add or improve a language, see [Translations](../contribute/translations.md).
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

## Household (per-user)

Who you cook for. Each person has a name, **Allergies** (the EU's 14 allergens, which cover the US's nine), **Other Allergies** (any other food, such as corn or kiwi, as strict as an allergen but found by ingredient name only, since labels don't list them), a **Diet** (Vegetarian, Pescatarian, Vegan, Gluten Free, Dairy Free), **Dislikes** (foods, typed one at a time) and the **Home** days they eat with you; none picked means every day. The day toggles work right on each person's card, and the pencil opens the rest.

What it changes:

- Planned servings start at who's home that day: in Plan a Cook, in Build My Week, and when Trace plans a cook. A number you set by hand stays.
- Allergy and diet warnings name the person they're for ("Peanuts · Alex"). On a planned day they count only who's home that day. See [Allergens](recipes.md#allergens).
- Build My Week leaves out what's a problem for someone home that day and ranks lower what someone doesn't like.

The setup wizard asks the same question (**Who Do You Cook For?**), and you can skip it. The household syncs with your account like your other settings. Allergy warnings are a help, not a guarantee: they come from ingredient names and product labels, so always check the packaging.

## Nutrition (per-user)

- **Visible Nutriments**: pick which of the 31 nutriments in the catalog show inline on the Nutrition Facts box and the Pantry details sheet. Everything else still counts toward totals; this is a display filter.

## AI Assistant (per-user unless env-locked)

- **Enable Trace AI**.
- **Provider**: Anthropic Claude, OpenAI, Google Gemini, or OpenAI Compatible.
- **Model**: preset dropdown per provider, plus **Custom...** for free-text model IDs.
- **API Key**: saved server-side per user. Optional for OpenAI Compatible, for a server that needs none (Ollama, LM Studio).
- **Test Connection** has its own row (from the next release): run a test any time; the status line says why one failed. The setting you fill in last tests on its own once everything it needs is there.
- **Base URL** (Custom or OpenAI Compatible only): e.g. `http://ollama:11434`, with or without `/v1`, or the whole `/v1/chat/completions` address (from the next release).
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

Only present in the Capacitor build. Set or change the server URL, disconnect, or switch between local (offline) and connected modes. To connect, type the server's address: its SSO buttons appear, along with username and password when the server allows them. SSO here needs a CookTrace server from the next release; with an older one, connect with a password. If the app can't load the server's sign-in options it says so, with **Retry**. See [Mobile modes](../mobile/modes.md).

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
- **Scheduled Backup** (env-lockable): Cadence Off/Daily/Weekly/Monthly, Time of Day (on the clock of the browser that set it, with its time zone shown under it and the next run beside it, from the next release), Retention Count.
- **Portable JSON Export**: Export JSON (no images), Import JSON (merges).
- **Local Full Backup** (Android local mode only): self-contained zip with base64-inlined images for phone-to-phone transfer.
- **Local Backup Schedule** (Android): device-side, persisted in `localStorage`.
- **Danger Zone**: Clear all data, Clear all settings.

Deep dive: [Backups](../self-hosting/backups.md), [Scheduled backups](../self-hosting/scheduled-backups.md).

## Import from Another App

The bulk import card. Drop a Mealie / Mealime / Tandoor / Paprika archive or a CookTrace export. See [Import](import.md), [Migrate from Mealie](migrate-mealie.md).

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
