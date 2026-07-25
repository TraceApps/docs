# Trace in LiftTrace and Smart Log

Trace is the LiftTrace AI coach: a floating chat assistant with an animated face, docked bottom-right on every page. Two things separate it from a generic LLM chat window: it carries your live workout context on every request, and its FAB doubles as a hold-to-record voice logger that feeds the Smart Log parser.

Trace is provider-agnostic. Setup lives in the shared [Trace setup](../trace/setup.md) page; this page covers what Trace specifically knows and does inside LiftTrace.

## Live workout context

Every message you send hits `POST /api/ai/chat`, which assembles a compact snapshot of your training state and prepends it to the conversation before dispatching to your chosen provider. The snapshot includes:

- **Active program** and its templates (name, goal, current week if a multi-week block is active).
- **Last 14 workouts**: exercise names, sets, weight, reps, RPE annotations. Warm-ups filtered out.
- **Top 10 personal records** across the whole library.
- **30-day body-weight trend** if you have logged body stats.
- **Weekly goal** and 4-week frequency.
- **Current streak**.
- **Today's coach prescription** if a trainer has assigned one for the current date.

The upshot: you can ask "how did my push day compare to last week" or "am I recovering enough between squat sessions" and Trace answers from data, not memory. The context is rebuilt fresh each turn so there is no stale snapshot problem.

Chat requests are capped at 60 messages of history and 200 KB of payload per turn to keep provider costs and token counts sane.

## The hold-to-log gesture

The Trace FAB is draggable and doubles as a voice trigger. Press and hold for **700 ms**:

- FAB turns red.
- The animated face swaps to a mic icon.
- A short beep fires plus (on Android) a haptic tap.
- Voice recording starts using the Web Speech API in the browser or the native recognizer on Android.

Release the FAB to commit; the transcript feeds the Smart Log parser and opens the Smart Add review sheet with your sets ready to confirm. Slide off more than **100 px** from the FAB's center to cancel: the FAB greys, releasing there aborts and nothing is logged. Under the 6-px movement threshold before the 700 ms mark the app treats the gesture as a drag instead, so you can reposition the FAB without arming the mic.

Voice recognition permission is browser-managed on the web and manifest-declared on Android (`RECORD_AUDIO` plus a `RecognitionService` intent query).

## FFT frequency visualizer

While the Radio player is streaming music, Trace's ring turns into a **32-bar SVG frequency visualizer** that reacts in real time to the audio.

- On the web it runs off a `WebAudio` `AnalyserNode` tapped from the shared audio graph.
- On Android it runs off `FftAudioProcessor.java`, an ExoPlayer PCM tap that emits FFT frames to JavaScript through a Capacitor plugin. A separate RAF loop lerps the bars toward the target values so the animation stays smooth even when frames arrive irregularly.

Purely cosmetic; it does not affect playback. If you find it distracting, disable Trace's music-reactive mode from Settings > Integrations > Trace.

## Quick-ask suggestion chips

The chat opens with a row of context-aware chips: recent workouts, current program, PR queries, form checks. Tap one to pre-fill the input and hit send. The chip set rotates so you are not staring at the same six prompts every day.

## Sample prompts

Prompts that get the most out of the live-context injection:

```text
Give me a warmup ramp for my top set of squats today.
How did my push day last Thursday compare to the one before it?
Where am I plateauing?
Suggest a superset partner for my second bench day this week.
Is my volume balanced across chest / back / legs this month?
I felt gassed on set 3 of squats. Was my top weight too heavy for RPE 8?
Recommend a deload for next week based on my last block.
```

For form checks, attach a photo (camera or gallery button on the chat input). Multimodal-capable providers (Claude, GPT-4-class OpenAI models, Gemini) will actually look at it.

## Smart Log entry points

Smart Log is the parser that turns natural-language workouts into structured sets. Three ways to get to it:

1. **Smart Add** button in the Diary toolbar. Type or paste.
2. **Sparkle FAB** on the diary (mobile).
3. **Hold-to-record** on the Trace FAB (700 ms hold, described above).

Parser coverage: cross-multiplied sets (`3x5 @ 225`), RPE targets (`RPE 8`), rep ranges (`6-8`), AMRAP, bodyweight (`@ BW`), supersets (`A1:` / `A2:`), and short aliases (`BB`, `DB`, `BP`, `OHP`, `DL`, `SQ`, `RDL`). Fuzzy name matching against your library: exact, starts-with, substring, token overlap. See [Diary and set logging](diary.md#smart-add-and-the-natural-language-parser) for the full syntax.

Preview always runs first. Fix names in place, delete rows you did not mean, then commit. Nothing is written to the diary until you confirm.

## Provider setup

Trace supports Claude, OpenAI, Gemini, and any OpenAI-compatible endpoint (Ollama, LM Studio, LocalAI, vLLM, DeepSeek, Groq, llama.cpp, Mistral). Setup lives in Settings > Integrations > Trace: pick a provider, enter a key, pick a model, optionally rename the assistant.

Two modes:

- **Per-user key** (default): every user brings their own key. Nothing to set server-side.
- **Server-side proxy**: set `AI_PROVIDER`, `AI_API_KEY`, and (for `oai-compat`) `AI_BASE_URL` and `AI_MODEL` in the server env, and Trace becomes read-only for every user. The server proxies chat requests, so a private-network LLM works without exposing it to the browser.

The Model dropdown includes a **Custom** option on every provider (added in v1.0.1), so you can enter any model ID the vendor supports without waiting for the preset list to catch up.

Retired Gemini `1.5-*` and `2.0-*` model IDs are silently remapped to the current default at request time (v1.0.1), so saved chat history keeps working after Google retires a model.

Full provider setup with per-provider recipes: see [Trace setup](../trace/setup.md) and [Running a local LLM](../trace/local-llm.md).

## Related

- [Trace setup](../trace/setup.md)
- [Running a local LLM](../trace/local-llm.md)
- [Diary and set logging](diary.md)
