# Model list and Gemini retirement remap

Google retires Gemini models on a rolling schedule. An `AI_MODEL` value that was fine six months ago will 404 today against `generativelanguage.googleapis.com`. To keep Trace working across upgrades without forcing every self-hoster to re-edit their compose file, the server maintains a set of known-dead Gemini IDs and remaps them to the current default at call time.

## How the remap works

Each app keeps the same three structures in two places, the app (`src/lib/aiChat.js`) and the server (`server/routes/ai.js`), and a test keeps the copies identical. NutriTrace 1.4.0 and NoteTrace 1.1.0 and later:

```js
const AI_DEFAULT_MODELS = {
  claude: 'claude-haiku-5-5',
  openai: 'gpt-5.6-luna',
  gemini: 'gemini-3.8-flash',
};

// Model IDs that changed name: a saved old name is sent as the new one.
const AI_MODEL_RENAMES = {
  'gemini-3.1-pro': 'gemini-3.1-pro-preview',
  'gemini-3-pro-preview': 'gemini-3.1-pro-preview',
  'gpt-5.6': 'gpt-5.6-sol',
};

// Models Google has shut down: sent as the default instead.
const GEMINI_RETIRED = new Set([ /* see the list below */ ]);
```

A saved config pointing at `gemini-2.0-flash` silently runs against `gemini-3.8-flash` instead, and one pointing at `gpt-5.6` runs against `gpt-5.6-sol`. Nothing changes in your `app_config` row or `AI_MODEL` env var; the swap happens per request. It applies on every path: a personal key calling the provider from the browser or the app, and the server's own proxy, relay and voice-note transcription when Trace is set by env vars. OpenAI-compatible endpoints (`oai-compat`) are never remapped: they name their own models. In **Settings, Trace**, a renamed model shows under its new name.

CookTrace and LiftTrace get the same lists in their next release; until then they remap only the four Gemini IDs from the 1.5 and 2.0 generations.

## Currently retired Gemini IDs

Shut down per [Google's deprecations page](https://ai.google.dev/gemini-api/docs/deprecations), as of 2026-10-11:

- `gemini-1.5-flash`, `gemini-1.5-pro`
- `gemini-2.0-flash`, `gemini-2.0-flash-001`, `gemini-2.0-flash-lite`, `gemini-2.0-flash-lite-001`, `gemini-2.0-flash-lite-preview`, `gemini-2.0-flash-lite-preview-02-05`
- `gemini-2.5-pro-preview-03-25`, `gemini-2.5-pro-preview-05-06`, `gemini-2.5-pro-preview-06-05`
- `gemini-2.5-flash-preview-05-20`, `gemini-2.5-flash-preview-09-25`, `gemini-2.5-flash-lite-preview-09-2025`
- `gemini-3.1-flash-lite-preview`

If Google retires more, the set is updated in a normal release cycle. Watch each app's changelog for the entry.

## Overriding the default with a custom model

Two ways.

**In the UI**: `Settings → Trace AI → Model → Custom…`. A free-text input appears. Type any model ID your provider accepts. It's stored verbatim in `user_settings.aiModel` (per-user mode) or `app_config.ai_model` (env-locked, if the admin picked it that way).

**In env**: set `AI_MODEL` to any string the provider accepts. There is no `AI_MODEL_CUSTOM` variable; the sentinel `__custom__` is only meaningful inside the Svelte picker. When you set `AI_MODEL=claude-sonnet-5-5` (or any other real ID), that's what the server uses.

If you override to a Gemini ID that's later added to `GEMINI_RETIRED`, the remap will silently start applying. Claude has no retirement set, and OpenAI only the renames above.

## Current defaults per provider

| Provider | NutriTrace 1.4.0, NoteTrace 1.1.0 and later | CookTrace, LiftTrace (until their next release) |
| -------- | ------------------------------------------- | ----------------------------------------------- |
| Claude   | `claude-haiku-5-5`                          | `claude-haiku-4-5-20251001`                     |
| OpenAI   | `gpt-5.6-luna`                              | `gpt-5.6-luna`                                  |
| Gemini   | `gemini-3.8-flash`                          | `gemini-3.6-flash`                              |
| oai-compat | (no default: `AI_MODEL` required)         | (no default: `AI_MODEL` required)               |

## Related

- [Setting up Trace](setup.md)
- [Cloud providers](cloud.md)
- [Local LLMs](local-llm.md)
