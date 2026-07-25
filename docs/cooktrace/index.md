# CookTrace

CookTrace is a self-hosted recipe box, pantry inventory, cook diary, meal planner, and shopping list bundled into a single Docker container. One Node/Express server, one Svelte PWA, a SQLite database, and a Capacitor Android app that either runs fully offline or syncs against the same server. AGPL-3.0, no accounts phoning home, no per-device license fee.

The core idea: every ingredient a recipe uses can link to a real pantry item, so cards on the Recipes page show "8/10 in pantry" at a glance, the shopping list dedups across recipes automatically, and expiration digests know what's about to go bad. Trace, the same AI assistant the other TraceApps use, sits on top with 19 cooking-domain tools so you can ask "what can I make from what's in the pantry?" and get real answers backed by your own data.

![CookTrace recipe library with pantry-match pills, ratings, times, and search](../assets/img/cooktrace/01-recipes.png)

## Compared to other self-hosted recipe apps

CookTrace is not "better than" the other options, just different in what it prioritises.

- **Mealie** is a mature recipe manager with an excellent import surface. CookTrace matches the import surface (schema.org JSON-LD, `recipe-scrapers` bridge, Mealie/Tandoor/Paprika zip drop-in) and adds a first-class pantry that every ingredient can link to, plus Trace AI tool use and OIDC SSO out of the box.
- **Paprika** is a polished commercial app you pay per device for. CookTrace covers most of the same daily flow (recipe library, cook mode, meal planner, grocery list) as a self-hosted single-container option, with Pinterest-style card share and long-press menus that feel similar.
- **Grocy** owns household inventory (barcodes, expiration, batteries, chores, meal plans). CookTrace overlaps only on the food side: pantry variants, barcode scanning, expiration digests. If you want chore lists and battery tracking too, Grocy is still the right pick.

## What's inside

- Recipe library with cook mode, live scaling, per-step photos, and an FDA-style Nutrition Facts box.
- Pantry with variants, expiration digests, barcode scanning, and Open Food Facts + USDA lookups.
- Cook Diary and Meal Planner sharing a calendar view, one tap turns a planned cook into a logged cook.
- Shopping list built from a recipe or a whole week's plan, grouped by aisle, dedupped across recipes.
- Kitchens for soft user groups with auto-share, plus per-user recipe grants and public share links.
- Trace AI assistant with 19 cooking tools; bring your own key or point at a local Ollama.
- URL / file / photo import in three tiers (Standard, Enhanced, Smart) so most sources land cleanly.
- Full OIDC SSO with 6 preset IdPs, plus classic email/password with optional zxcvbn strength policy.
- Android app that runs fully offline or syncs against your server.

## Get started

- [Compose and first boot](../getting-started/compose.md) walks through the Docker install.
- [Feature tour](features.md) is the guided walkthrough if you'd rather see what's in the app before installing.
- [Env vars](env-vars.md) and [Settings reference](settings.md) cover the two big configuration surfaces.

## Related

- [Trace AI setup](../trace/setup.md)
- [Mobile install](../mobile/install.md)
- [Self-hosting notes](../self-hosting/env-vars.md)
