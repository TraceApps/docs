# Translations (i18n)

All three apps use [svelte-i18n](https://github.com/kaisermann/svelte-i18n) with one JSON file per locale in `src/i18n/`. English (`en.json`) is the source of truth; every other locale is a translated copy with the same key structure. The runtime falls back to English for any missing key, so a partial translation is safe to release.

## Two ways to contribute a translation

Both end up with the same file (`src/i18n/<code>.json`) in the repo. Pick whichever suits your situation.

### A. Weblate (preferred for ongoing work)

[Weblate](https://weblate.org/) is a browser-based translation platform. You pick a language, translate strings inline, and commits go back to `dev` as automatic pull requests. No git, no JSON syntax, no code.

Weblate projects per app:

| App | Weblate project |
|-----|-----------------|
| CookTrace  | [hosted.weblate.org/projects/cooktrace/](https://hosted.weblate.org/projects/cooktrace/) |
| LiftTrace  | [hosted.weblate.org/projects/lifttrace/](https://hosted.weblate.org/projects/lifttrace/) |
| NutriTrace | [hosted.weblate.org/projects/nutritrace/](https://hosted.weblate.org/projects/nutritrace/) |

Flow:

1. Open the Weblate project page. If your language is already in the list, click into it and start translating.
2. If your language isn't in the list, click **Add new translation**, pick the language, and the project maintainer will approve it (usually same day).
3. If you have an existing translation file (from a prior PR, an old app fork, or another project), seed the language with it: on the language page, **Files → Upload translation → attach the JSON, pick "Add as translation"**. Weblate imports every matching key and marks the rest for you to fill in.
4. Translate remaining strings through the web UI. Weblate handles the git side automatically and periodically opens PRs to `dev`.

**One gotcha**: once Weblate is watching a language, direct edits to that file in a PR get overwritten on the next Weblate sync (Weblate holds the authoritative state in its own database). Use Weblate for edits going forward once a language is on it.

### B. Direct JSON PR (fine for a one-shot contribution)

If you have a completed translation file you'd rather submit once and be done, or you prefer working locally, this path still works. Weblate is compatible with any file that already exists in the tree, it picks it up on the next sync (and then owns further edits per the gotcha above).

Follow the "Adding a new language" steps below.

Current locale coverage (2026-08-21):

| App | Locales in tree |
|-----|-----------------|
| CookTrace  | `en`, `sv` |
| LiftTrace  | `en` |
| NutriTrace | `en`, `fr` |

## Adding a new language (direct PR path)

1. Copy `src/i18n/en.json` to `src/i18n/<code>.json` where `<code>` is a BCP-47 short code (`fr`, `de`, `nl`, `es`, `pt`, `pt-BR`, `ja`, `zh_Hans`, `zh_Hant`, etc.). Translate the values. Leave the keys untouched. HTML and Markdown inside values (`<strong>`, `<br>`, etc.) stays as-is.

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

    It reports missing keys, orphaned keys (present in your locale file but not in `en.json`), and, on stricter runs, keys where the value matches the English source (usually a sign of copy-paste without translation). Aim for 100 percent coverage on first submit; if new English strings are added during the release cycle before you finish, the delta is small and the fallback keeps the app usable.

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

## Related

- [Dev setup (all three)](dev-setup.md)
- [Changelogs](../reference/changelogs.md)
