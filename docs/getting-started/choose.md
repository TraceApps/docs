# Choose your app

TraceApps is three self-hosted trackers built by the same author, sharing the same design language, the same "Trace" assistant, and the same install story. Pick whichever fits what you want to track. Nothing stops you running all three side by side on one box; they each get their own container, their own database, and their own port.

If you already know which app you want, jump straight into [Install with Docker Compose](compose.md).

## CookTrace

For cooking. Recipes, a pantry that tracks what you actually have on hand, a cook diary of what you've made, a shopping list that flows out of your meal plan, and cookbooks you can group recipes into. Import recipes from 300+ sites, a photo of a cookbook page, or a URL. Trace can log a cook, add items to the pantry, plan a week, and suggest what to make from what's in stock.

Best fit if you want a Mealie-adjacent kitchen brain that stays 100% yours.

Start here: [CookTrace overview](../cooktrace/index.md).

## LiftTrace

For lifting. Workout diary with per-set logging, programs and templates, multi-week progression, a full exercise library seeded from wger, personal records, and a coaching mode for training clients. Bonus: a built-in Radio player (Subsonic/Navidrome and Icecast/SHOUTcast streams) so the app doubles as your gym-session player. Trace's Smart Log parses "bench 3x5 @ 225" in plain text or voice.

Best fit if you're done wrestling with Hevy or Strong and want your set history on your own hardware.

Start here: [LiftTrace overview](../lifttrace/index.md).

## NutriTrace

For nutrition. Food diary with barcode scan and photo-based Scan Label, an adaptive TDEE that adjusts to how you actually move and eat, a Wellness tab that pulls from Fitbit, Google Health, Withings, Garmin, and Health Connect, water and fasting tracking, manual activity logging, and goal celebrations that fire once (not every login). Full Open Food Facts integration with an optional local DuckDB mirror for air-gapped setups.

Best fit if you're leaving MyFitnessPal, Cronometer, Lose It, or Waistline and want an offline-friendly replacement that also reads your wearables.

Start here: [NutriTrace overview](../nutritrace/index.md).

## Shared plumbing

All three install the same way, use the same env vars, share the same OIDC and SMTP setup, and the same Docker tag scheme. Configure once, apply everywhere:

- [Install with Docker Compose](compose.md)
- [First-run wizard](first-run.md)
- [Reverse proxy](reverse-proxy.md)
- [Updating](updating.md)
