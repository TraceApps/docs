# Model list and Gemini retirement remap

Google retires Gemini models on a rolling schedule. An `AI_MODEL` value that was fine six months ago will 404 today against `generativelanguage.googleapis.com`. To keep Trace working across upgrades without forcing every self-hoster to re-edit their compose file, the server maintains a set of known-dead Gemini IDs and remaps them to the current default at call time.

## How the remap works

In `server/routes/ai.js`, the server keeps two structures:

```js
const AI_DEFAULT_MODELS = {
  claude: 'claude-haiku-4-5-20251001',
  openai: 'gpt-4o-mini',
  gemini: 'gemini-2.5-flash',
};

const GEMINI_RETIRED = new Set([
  'gemini-1.5-flash', 'gemini-1.5-pro',
  'gemini-2.0-flash', 'gemini-2.0-flash-lite',
]);
```

Inside the Gemini caller:

```js
const m = GEMINI_RETIRED.has(model) ? AI_DEFAULT_MODELS.gemini : (model || AI_DEFAULT_MODELS.gemini);
```

A saved config pointing at `gemini-1.5-flash` will silently execute against `gemini-2.5-flash` instead. Nothing changes in your `app_config` row or `AI_MODEL` env var; the swap happens per request. This applies only to the env-locked server proxy path (`POST /api/ai/chat`); per-user browser-direct calls do not get the remap because the browser talks straight to Google, so a browser-direct request against a retired ID will still 404. The recommended fix in that case is to pick a current model in `Settings → Trace AI`.

## Currently retired Gemini IDs

As of the last audit sweep:

- `gemini-1.5-flash`
- `gemini-1.5-pro`
- `gemini-2.0-flash`
- `gemini-2.0-flash-lite`

If Google retires more, the set is updated as part of a normal release cycle (the family memory `feedback_traceapps_ai_model_freshness` covers the sweep cadence). Watch [the CookTrace changelog](https://github.com/TraceApps/cooktrace/blob/main/CHANGELOG.md), [LiftTrace changelog](https://github.com/TraceApps/lifttrace/blob/main/CHANGELOG.md), or [NutriTrace changelog](https://github.com/TraceApps/nutritrace/blob/main/CHANGELOG.md) for the entry.

## Overriding the default with a custom model

Two ways.

**In the UI**: `Settings → Trace AI → Model → Custom…`. A free-text input appears. Type any model ID your provider accepts. It's stored verbatim in `user_settings.aiModel` (per-user mode) or `app_config.ai_model` (env-locked, if the admin picked it that way).

**In env**: set `AI_MODEL` to any string the provider accepts. There is no `AI_MODEL_CUSTOM` variable; the sentinel `__custom__` is only meaningful inside the Svelte picker. When you set `AI_MODEL=claude-sonnet-5` (or any other real ID), that's what the server uses.

If you override to a Gemini ID that's later added to `GEMINI_RETIRED`, the remap will silently start applying. If you want to force a specific alpha or preview model and skip the remap, use Claude or OpenAI (which don't have a retirement set) or roll your own build.

## Current defaults per provider

| Provider | Default `AI_MODEL`               |
| -------- | -------------------------------- |
| Claude   | `claude-haiku-4-5-20251001`      |
| OpenAI   | `gpt-4o-mini`                    |
| Gemini   | `gemini-2.5-flash`               |
| oai-compat | (no default: `AI_MODEL` required) |

## Related

- [Setting up Trace](setup.md)
- [Cloud providers](cloud.md)
- [Local LLMs](local-llm.md)
