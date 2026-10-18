# Local LLMs

Trace treats any OpenAI-compatible endpoint (Ollama, LM Studio, vLLM, LocalAI, DeepSeek, Groq, etc.) as one provider: `oai-compat`. If your server can serve `POST /v1/chat/completions`, Trace can talk to it.

## The three settings

Pick `OpenAI Compatible` from the provider dropdown, or set:

```env
AI_PROVIDER=oai-compat
AI_BASE_URL=http://ollama:11434
AI_MODEL=llama3.1:8b
# AI_API_KEY is optional; leave empty for Ollama and most local runners.
```

- `AI_BASE_URL` is required. It's the root of the endpoint, such as `http://ollama:11434`; the app adds `/v1/chat/completions` itself. LiftTrace 1.4.0 and later also take it with `/v1` on the end, or as the whole `/v1/chat/completions` address, in Settings and in `AI_BASE_URL`. The other apps do from their next release; until then, leave `/v1` off, or every request goes to `/v1/v1/chat/completions`.
- `AI_MODEL` is required. Local runners don't have a "default" model to fall back to, so the request has to name one exactly.
- `AI_API_KEY` is optional. Ollama and LocalAI don't require one; the client sends `no-key` internally so the `Authorization` header stays valid. Set a real key only if your runner is behind a reverse-proxy that checks bearer tokens.

## Docker compose sidecar

The tidiest setup is to run Ollama in the same compose file as the Trace app. Both services live on the same internal Docker network, so `AI_BASE_URL=http://ollama:11434` resolves by service name without exposing Ollama to the host.

```yaml
services:
  cooktrace:
    image: ghcr.io/traceapps/cooktrace:latest
    ports:
      - "3003:3003"
    volumes:
      - ./data/db:/data/db
      - ./data/uploads:/data/uploads
    environment:
      JWT_SECRET: ${JWT_SECRET}
      AI_PROVIDER: oai-compat
      AI_BASE_URL: http://ollama:11434
      AI_MODEL: llama3.1:8b
    depends_on:
      - ollama

  ollama:
    image: ollama/ollama:latest
    volumes:
      - ./data/ollama:/root/.ollama
    # No `ports:` block: the model server is only reachable from inside
    # the compose network, not from the host. Trace talks to it over the
    # Docker DNS name `ollama`.
    # deploy: { resources: { reservations: { devices: [...] } } }
    #   Add an NVIDIA GPU reservation here if you have one; CPU-only
    #   inference works but is slow.
```

Pull a model into the sidecar once:

```bash
docker compose exec ollama ollama pull llama3.1:8b
```

Any model you pull sticks around in `./data/ollama`, so the pull only happens once per compose stack.

The same shape works for LM Studio (start it with the server on, use `AI_BASE_URL=http://host.docker.internal:1234`), vLLM, or LocalAI. Trace does not care which runner it is, only that the wire shape matches OpenAI's.

## Model size guidance

Local model choice is a real trade-off; the app doesn't hide it. A rough guide:

- **7-8B parameters** (Llama 3.1 8B, Qwen 2.5 7B, Mistral 7B): fine for chat, natural-language logging, summaries. Runs on a modern laptop CPU in a pinch, and comfortably on an 8 GB GPU. Tool-use quality is inconsistent; Trace's tool loop will sometimes give up or repeat itself.
- **13-14B** (Qwen 2.5 14B, Phi 4): the practical minimum for reliable tool use across Trace's schemas. Needs a 12 GB+ GPU for a good experience.
- **32B+** (Qwen 2.5 32B, Llama 3.3 70B via a smaller quant): tool-use quality starts to approach GPT-4o-mini. Realistically wants a 24 GB GPU or a serious workstation.

If Trace's tools misbehave on a small local model, look at `feedback_ai_unreliable_tool_routing` in the CookTrace / LiftTrace / NutriTrace repos: the pattern is to pre-compute the value in `buildContext()` and inject it as a labeled system-prompt line rather than counting on the model to route a tool call correctly.

!!! warning "Slow first response"
    Ollama loads the model into VRAM on the first request after startup or after a period of inactivity. That first Trace turn can take 20-60 seconds while the weights are read from disk. Subsequent turns are fast until the model is unloaded again. This is Ollama behaviour, not a Trace bug.

## Testing the connection

In LiftTrace 1.4.0 and later (the other apps from their next release), **Test Connection** has its own row in the AI settings, so you can run a test at any time, also against an OpenAI Compatible server that needs no API key. The setting you fill in last tests on its own once everything it needs is there, and a test only counts for the settings it tested.

## Models without tool support

Some local models can't call tools, and the runner answers with an error such as `tool use is not supported`. NoteTrace 1.1.0 and LiftTrace 1.4.0 and later then ask again without tools and say once that the model can't use them: Trace still answers, but can't look anything up or log anything. NutriTrace and CookTrace do the same from their next release; until then every chat with such a model fails. For Trace's full set, pick a model with tool support (in Ollama's library, one tagged **tools**).

## When Trace gets no answer

With a per-user setup, the browser calls your provider itself, so the provider has to allow requests from the app's address (CORS) on every answer, errors included. A reverse proxy that adds those headers only to successful answers hides the provider's real error, such as a model that can't use tools, behind a bare "Failed to fetch". LiftTrace 1.4.0 and later (the other apps from their next release) say what to check instead:

- the **Base URL**;
- that the server allows requests from the app's address on every answer, errors included (with nginx, `add_header ... always`);
- on a page served over HTTPS, an `https://` address for the provider, since the browser won't call a plain `http://` one.

Setting Trace up on the server instead (`AI_PROVIDER`, `AI_BASE_URL`, `AI_MODEL`) avoids all three: the server calls the provider, not the browser.

## Long replies and reasoning models

A reasoning model (Qwen3, DeepSeek-R1 and others) thinks before it answers, and the thinking counts toward the reply's length. Trace used to ask for at most 4,096 tokens, which a long rewrite or a lot of thinking could use up, leaving no answer. From each app's next release, Trace asks for up to 16,384 tokens, with every provider ([notetrace#23](https://github.com/TraceApps/notetrace/issues/23)). A server or model that won't take that many, such as vLLM before 0.32 serving a small context, is asked again with 4,096 automatically.

A reply that still stops at its limit, or one with reasoning but no answer, is never used: Trace says so instead of saving half a rewrite or running half a tool call. If that keeps happening, ask for less at once, turn the model's reasoning down, or give the server a larger context.

## Streaming and timeouts

Trace does not use streaming: it waits for the full response, then renders. Every request asks for a single answer (`stream: false`), so an endpoint that streams unless told not to works too (NutriTrace 1.4.0, NoteTrace 1.1.0 and LiftTrace 1.4.0 and later; CookTrace from its next release). If your local model is very slow (or generating a very long reply), the browser fetch times out at the provider level rather than Trace's. When Trace is set by environment variables, the server waits up to 5 minutes for a reply (from each app's next release). Bump `AI_BASE_URL` to a runner with faster inference, or pick a smaller model.

## Related

- [Setting up Trace](setup.md)
- [Cloud providers](cloud.md)
- [Model list and retirement remap](models.md)
