# Translations (i18n)

All three apps use [svelte-i18n](https://github.com/kaisermann/svelte-i18n) with one JSON file per locale in `src/i18n/`. English (`en.json`) is the source of truth; every other locale is a translated copy with the same key structure. The runtime falls back to English for any missing key, so a partial translation is safe to release.

## Preferred: translate on Weblate (no code required)

**NutriTrace** is hosted on [Weblate](https://weblate.org/) — a browser-based platform for libre translation projects. Pick a language, translate strings inline, and commits land as pull requests automatically. No git, no JSON syntax, no code:

- [hosted.weblate.org/projects/nutritrace/](https://hosted.weblate.org/projects/nutritrace/)

CookTrace and LiftTrace aren't hosted on Weblate yet — for those, follow the JSON-file flow below.

Current locale coverage (2026-07-25):

| App | Locales in tree |
|-----|-----------------|
| CookTrace  | `en`, `sv` |
| LiftTrace  | `en` |
| NutriTrace | `en`, `fr` |

## Adding a new language

1. Copy `src/i18n/en.json` to `src/i18n/<code>.json` where `<code>` is a BCP-47 short code (`fr`, `de`, `nl`, `es`, `pt`, `pt-BR`, `ja`, etc.). Translate the values. Leave the keys untouched. HTML and Markdown inside values (`<strong>`, `<br>`, etc.) stays as-is.

2. Register the locale in `src/i18n/index.js`. Add a `register()` call and append the locale to `AVAILABLE_LOCALES`:

    ```js
    import { register, init, getLocaleFromNavigator } from 'svelte-i18n';

    register('en', () => import('./en.json'));
    register('fr', () => import('./fr.json'));  // your new line

    export const AVAILABLE_LOCALES = [
      { code: 'en', label: 'English' },
      { code: 'fr', label: 'Français' },  // your new line
    ];
    ```

    Without this step the JSON sits in the repo but the language picker in Settings cannot surface it.

3. Run the lint script to confirm coverage:

    ```bash
    npm run i18n:check
    ```

    It reports missing keys, orphaned keys (present in your locale file but not in `en.json`), and, on stricter runs, keys where the value matches the English source (usually a sign of copy-paste without translation). Aim for 100 percent coverage on first submit; if the release cycle lands new English strings before you are done, the delta is small and the fallback keeps the app usable.

4. Test locally. Start the app in dev mode, open **Settings > Regional > Language**, pick your locale, and walk the main screens. Watch for strings that overflow buttons or wrap awkwardly in narrow columns.

5. Open a PR against `dev` with the two changed files (`src/i18n/<code>.json` and `src/i18n/index.js`). Translator authorship is preserved on the JSON file's commit history; do not reformat or reorder the English file.

## For code contributors: instrumenting new strings

Every user-facing string added to any of the three apps should be extracted into `en.json` and rendered through svelte-i18n's `$_()` helper. Hardcoded English literals in templates are the reason translation coverage lags the codebase; prevent them at PR time rather than retrofit later.

The pattern:

```svelte
<script>
  import { _ } from 'svelte-i18n';
</script>

<button>{$_('diary.add_food')}</button>
```

Then in `src/i18n/en.json`:

```json
{
  "diary": {
    "add_food": "Add food"
  }
}
```

Two rules that catch community PRs:

- **Only add English** in your PR. Do not machine-translate into languages you do not natively speak; that misrepresents contributor work and makes the resulting locale file harder to correct. The `en.json` addition is enough; native speakers fill in their locale files in follow-up PRs.
- **Run `npm run i18n:check`** before opening the PR. It catches orphaned keys, typos, and stale entries. CI does not gate on it yet, but merging a PR that breaks the check is a minor annoyance.

## Proper nouns: do not translate

Product names, brand names, and third-party service names stay in English regardless of target locale:

Trace, Trace AI, CookTrace, LiftTrace, NutriTrace, TraceApps, wger, USDA, Open Food Facts, OFF, Mealie, Fitbit, Google Health, Withings, Garmin, Health Connect, Ollama, LM Studio, LocalAI, vLLM, DeepSeek, Groq, Together AI, Mistral, Anthropic, Claude, OpenAI, GPT, Google, Gemini, Authentik, Keycloak, Pocket-ID, Authelia, Auth0, Apprise, Gotify, ntfy, Docker, Caddy, Traefik, nginx, Cloudflare, Tailscale, Capacitor, Svelte, Vite, Node.js, Express, SQLite, DuckDB, JWT, OIDC, SSO.

Feature names inside the apps (Smart Log, Quick Calories, Adaptive TDEE, Meal Planner, Kitchen Gear, Cook Diary, Programs, Radio) also stay in English, matching the UI label.

## Weblate

The project structure is Weblate-ready (per-locale JSON with a stable key layout), but there is no hosted Weblate instance yet. Translations happen through PRs against the app repos. If a translator community forms around any of the apps, a Weblate mirror is the natural next step.

## Related

- [Dev setup (all three)](dev-setup.md)
- [Changelogs](../reference/changelogs.md)
