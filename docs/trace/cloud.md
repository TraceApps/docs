# Cloud providers

Three cloud providers are wired in: Anthropic Claude, OpenAI, and Google Gemini. This page covers where to get an API key and which model each app defaults to.

The setup flow is the same for all three: `Settings → Trace AI`, pick the provider, paste the key, pick a model, Save. For the env-locked variant (one config for everyone), see [Setting up Trace](setup.md).

## Anthropic Claude

Get a key from the [Anthropic Console](https://console.anthropic.com/settings/keys). Free tier plus prepaid credit, both work.

Default model: `claude-haiku-4-5-20251001` (fast, cheap, strong tool-use).

Picker also includes `claude-sonnet-5` (smarter, roughly 3-5× the cost) and `claude-opus-4-8` (smartest, roughly 15× the cost). Sonnet is the sweet spot for most people once cost stops being the top concern.

Env config:

```env
AI_PROVIDER=claude
AI_API_KEY=sk-ant-api03-...
AI_MODEL=claude-haiku-4-5-20251001
```

Wire endpoint is `https://api.anthropic.com/v1/messages` with header `anthropic-version: 2023-06-01`. Claude has the richest native tool-use, which is why it's the default across all three apps.

## OpenAI

Get a key from the [OpenAI platform dashboard](https://platform.openai.com/api-keys). Prepaid credit required (no free tier).

Default model: `gpt-4o-mini` (fast, cheap).

Picker also includes `gpt-4o` (smarter, roughly 15× the cost of mini). If you want a newer model like `gpt-4.1` or `o3`, use `Custom…` in the picker and type the ID, or set `AI_MODEL` directly.

Env config:

```env
AI_PROVIDER=openai
AI_API_KEY=sk-proj-...
AI_MODEL=gpt-4o-mini
```

Wire endpoint is `https://api.openai.com/v1/chat/completions`. The same code path powers `oai-compat` against Ollama and friends; only the base URL differs.

## Google Gemini

Get a key from [Google AI Studio](https://aistudio.google.com/app/apikey). There's a generous free tier, which makes Gemini a good "just try Trace" option.

Default model: `gemini-2.5-flash` (fast, cheap, strong).

Picker also includes `gemini-2.5-flash-lite` (cheapest) and `gemini-2.5-pro` (smarter). Any earlier Gemini model (`1.5-flash`, `1.5-pro`, `2.0-flash`, `2.0-flash-lite`) has been retired by Google and will 404 if called directly; the server auto-remaps saved configs pointing at those IDs to the current default. See [Model list and retirement remap](models.md).

Env config:

```env
AI_PROVIDER=gemini
AI_API_KEY=AIzaSy...
AI_MODEL=gemini-2.5-flash
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
