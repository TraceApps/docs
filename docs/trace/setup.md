# Setting up Trace

Trace is the in-app AI assistant baked into CookTrace, LiftTrace, and NutriTrace. It talks to a Large Language Model of your choice (cloud or self-hosted) and, in each app, it can also read and write real data: log a cook, add to your shopping list, parse `bench 3x5 @ 225` into a set, propose Quick Calories from a meal photo, and so on.

Before Trace does anything, someone has to point it at a provider and (for cloud providers) supply an API key. There are two ways to do that.

## Two setup modes

### Per-user

The default. Each user opens `Settings → Trace AI`, chooses a provider, pastes their own API key, picks a model, and hits Save. The key is stored in `user_settings` in the app's SQLite DB, keyed to their user id. It never mixes with another user's key. If the admin has done nothing at all, every user is on their own to configure Trace.

Good for: personal instances, family instances where each person prefers a different provider, or trial setups where the admin doesn't want to front the API bill for everyone.

### Env-locked (admin sets it once)

If any of `AI_PROVIDER / AI_API_KEY / AI_MODEL / AI_BASE_URL / AI_ENABLED` is present as an environment variable at server boot, that config is seeded into `app_config`, marked with `ai_env_locked=true`, and the per-user Settings fields freeze with a "Configured via environment variables" banner. All users then share the admin's config, and the API key never reaches the browser: the client sends only `{ messages, systemPrompt, tools? }` to `POST /api/ai/chat`, and the server proxies the request to Claude, OpenAI, Gemini, or your local endpoint.

Good for: multi-user instances where the admin wants one budget, one provider, and no risk of a user leaking a personal key into logs.

Remove the env vars and restart, and the lock clears on the next boot (fix for NT #36); the previously-seeded values stay in `app_config` but users can edit them again.

Either mode uses the same four-way provider switch and the same wire shapes. Only the storage location and the "who gets to change it" question differ.

!!! tip "Use Docker secrets"
    `AI_API_KEY_FILE=/run/secrets/ai_api_key` works anywhere `AI_API_KEY` does, so you can keep the raw key out of `docker-compose.yml`.

## The four providers

The provider dropdown is the same in every Trace app:

| Provider      | `AI_PROVIDER` value | Notes                                                                                       |
| ------------- | ------------------- | ------------------------------------------------------------------------------------------- |
| Claude        | `claude`            | Anthropic API. Best tool-use quality.                                                       |
| OpenAI        | `openai`            | GPT-4o family.                                                                              |
| Gemini        | `gemini`            | Google AI Studio API.                                                                       |
| OpenAI-compat | `oai-compat`        | Ollama, LM Studio, vLLM, LocalAI, DeepSeek, Groq, anything speaking the OpenAI `/v1/chat/completions` shape. |

Cloud providers (Claude / OpenAI / Gemini) require an API key. `oai-compat` usually does not; against Ollama and friends, the client sends `no-key` internally so the `Authorization` header stays valid.

Per-provider walkthroughs live on their own pages: see [Cloud providers](cloud.md) and [Local LLMs](local-llm.md).

## Picking a model

Each provider comes with a picker containing the current recommended defaults plus a `Custom…` entry (sentinel value `__custom__` in the UI). Choosing `Custom…` reveals a free-text field; whatever you type is stored verbatim in `ai_model` and used on every request. There is no separate `AI_MODEL_CUSTOM` env var: `AI_MODEL` accepts any string the provider recognises.

If you're env-locked and want a model outside the picker's list, set `AI_MODEL=your-model-id` in your compose file. The lock covers custom values too.

For Gemini specifically, retired model IDs are auto-remapped server-side so a stale config doesn't 404 against a shut-down endpoint. See [Model list and retirement remap](models.md).

## API key handling

- **Per-user mode**: the key is stored in `user_settings.aiApiKey` and sent from the browser directly to the provider's endpoint. The server never sees it.
- **Env-locked mode**: the key stays in `app_config.ai_api_key` (or the file referenced by `AI_API_KEY_FILE`) and never leaves the server. The browser posts to `/api/ai/chat`; the server does the outbound call.

Payload caps on the proxy bound a compromised account from burning your API budget: 60 messages per request, 200 KB per request in CookTrace and LiftTrace, 8 MB in NutriTrace (roomier because meal photos ride the same channel).

## Rate limiting

Every `/api/ai/chat` request goes through a per-user limiter: 30 requests per 60 seconds. Trace's own chat loop caps tool-use rounds at 5. Both numbers live in `server/routes/ai.js` if you want to change them.

## Related

- [Cloud providers](cloud.md)
- [Local LLMs](local-llm.md)
- [Model list and retirement remap](models.md)
