# Cloud providers

Three cloud providers are wired in: Anthropic Claude, OpenAI, and Google Gemini. This page covers where to get an API key and which model each app defaults to.

The setup flow is the same for all three: `Settings → Trace AI`, pick the provider, paste the key, pick a model, Save. For the env-locked variant (one config for everyone), see [Setting up Trace](setup.md).

The model lists were checked against each vendor's model and deprecation pages on 2026-10-11. NutriTrace 1.4.0 and NoteTrace 1.1.0 and later have the lists below; CookTrace and LiftTrace get them in their next release (until then their Claude default is `claude-haiku-4-5-20251001` and their Gemini default is `gemini-3.6-flash`). A model you already chose keeps working: one that isn't in the picker any more shows as a custom model.

## Anthropic Claude

Get a key from the [Anthropic Console](https://console.anthropic.com/settings/keys). Free tier plus prepaid credit, both work.

Default model: `claude-haiku-5-5` (fast, cheapest, strong tool use).

Picker also includes `claude-sonnet-5-5` (balanced), `claude-opus-5-5` (smarter) and `claude-fable-5-1` (most capable), plus the previous `claude-sonnet-5`, `claude-opus-5`, `claude-fable-5` and `claude-opus-4-8`. Sonnet is the sweet spot for most people once cost stops being the top concern.

Env config:

```env
AI_PROVIDER=claude
AI_API_KEY=sk-ant-api03-...
AI_MODEL=claude-haiku-5-5
```

Wire endpoint is `https://api.anthropic.com/v1/messages` with header `anthropic-version: 2023-06-01`. Claude has the richest native tool use, which is why it's the default in every app.

## OpenAI

Get a key from the [OpenAI platform dashboard](https://platform.openai.com/api-keys). Prepaid credit required (no free tier).

Default model: `gpt-5.6-luna` (fast, cheap).

Picker also includes `gpt-6-luna` (fast, cheapest), `gpt-5.6-terra` (balanced) and `gpt-5.6-sol` (most capable), plus the previous `gpt-4o-mini` and `gpt-4o`. GPT-6.1 Sol and GPT-6 Astra aren't listed: Trace calls tools through Chat Completions, which those two don't support. A saved `gpt-5.6` is sent as `gpt-5.6-sol`, the model it stands for.

Env config:

```env
AI_PROVIDER=openai
AI_API_KEY=sk-proj-...
AI_MODEL=gpt-5.6-luna
```

Wire endpoint is `https://api.openai.com/v1/chat/completions`. The same code path powers `oai-compat` against Ollama and friends; only the base URL differs.

## Google Gemini

Get a key from [Google AI Studio](https://aistudio.google.com/app/apikey). There's a generous free tier, which makes Gemini a good "just try Trace" option.

Default model: `gemini-3.8-flash` (fast).

Picker also includes `gemini-3.1-flash-lite` (cheapest), `gemini-3.5-flash-lite` (cheap) and `gemini-3.1-pro-preview` (most capable), plus the previous `gemini-3.6-flash`. Gemini 2.5 is open only to Google accounts that used it before, so it's no longer listed; a saved 2.5 model keeps working. A saved `gemini-3.1-pro` (an ID that never existed) is sent as `gemini-3.1-pro-preview`, and models Google has shut down are sent as the default. See [Model list and retirement remap](models.md).

Env config:

```env
AI_PROVIDER=gemini
AI_API_KEY=AIzaSy...
AI_MODEL=gemini-3.8-flash
```

Wire endpoint is `https://generativelanguage.googleapis.com/v1beta/models/<model>:generateContent`. Function-calling round-trips through OpenAI-shaped adapters on the server proxy, so Trace's full tool set works on Gemini too.

!!! warning "Photo uploads"
    Only NutriTrace routes meal photos through the AI channel by default (for `propose_food` and `propose_quick_calories`). LiftTrace and CookTrace don't send images by default, but the picker will accept an attachment if you post one manually. Every provider is happy with base64 `data:` URIs; the server proxy reshapes them per provider.

## Rate limits and cost

Trace itself is rate-limited to 30 requests / 60s per user (see [Setting up Trace](setup.md)). Beyond that, provider-side limits apply. All three providers surface a per-key spend dashboard; check yours before opening Trace up to a household.

## Related

- [Setting up Trace](setup.md)
- [Local LLMs](local-llm.md)
- [Model list and retirement remap](models.md)
