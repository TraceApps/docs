# Trace in NutriTrace + Smart Log

Trace is the in-app AI assistant. In NutriTrace it can read your diary, wellness, workouts, body composition, goals, and fasting history through tool use, and it can propose diary entries and new foods for you to review before anything is written. This page covers the NutriTrace-specific role of Trace: the tools it can call, the two propose-and-review flows, voice logging via Smart Log, and the label-scan flow. Provider setup (which LLM, which key, per-user vs env-locked) is shared with CookTrace and LiftTrace and lives on the [shared Trace setup page](../trace/setup.md).

## What Trace can do in NutriTrace

Trace runs a tool-use loop (up to 5 rounds per turn) with a set of NutriTrace-specific tools defined in `src/lib/aiChat.js`. The tools split into three groups.

### Read tools

- **`get_wellness_data`**: steps, calories out, sleep (duration, stages, score, efficiency), heart rate (RHR, HRV, SpO₂), respiratory rate, readiness, stress, skin temp, VO₂ max. Any date range. Sourced from Fitbit / Garmin / Health Connect.
- **`get_body_composition`**: weight, body fat percentage, muscle mass, bone mass, body water, lean/fat mass, visceral fat, vascular age, metabolic age, BMR, nerve health, ECG. Any date range. Sourced from Withings.
- **`get_diary`**: diary items (with brand, notes, per-item notes, day notes) for a date range.
- **`get_meals`**: the user's saved Meals library.
- **`get_workouts`**: workout list for a date range (wearable-synced and manual).
- **`get_activity_log`**: the manual activity log (workouts logged without a wearable).
- **`get_goals`**: the user's calorie and per-nutrient goal grid, plus the mode (Fixed / Dynamic / Adaptive).
- **`get_diary_averages`**: trailing averages of any tracked nutrient over N days.
- **`get_logging_streak`**: current and longest logging streak.
- **`get_fasting_history`**: completed and active fasts, streaks, longest.
- **`get_adaptive_tdee`**: the learned TDEE value and confidence percentage.

### Write tools

- **`log_food`**: add a specific known food to a meal on a date.
- **`log_quick_calories`**: add a Quick Calories entry (no persistent food row) to a meal on a date.
- **`add_activity_entry`**: log a manual workout.

### Propose-and-review tools

Two tools deliberately don't write anything on their own. They surface a review card in the chat with **Discard** / **Add to Diary** (or **Save to Foods**) buttons so you stay in control.

- **`propose_quick_calories`**: proposes a Quick Calories entry for you to confirm. Common trigger: attach a meal photo and ask Trace to estimate calories.
- **`propose_food`**: proposes a reusable food entry (with macros) that you can add to your Foods library, optionally logging it to the diary on the same tap.

These two tools have the strictest system-prompt guardrails. The model is told: calling this tool means "the card was shown", **not** "the food was logged". After calling it, Trace should not claim to have logged anything until you confirm.

## Sample prompts

Trace responds well to natural phrasing. Some prompts that exercise the tool set:

- "What's my average protein this week?"
- "How did I sleep last night?"
- "What was my last fast, and what's my streak?"
- "Log a large latte to breakfast."
- "I ate leftover chili for lunch. Half a serving." (matches your saved Meals)
- "Estimate the calories in this." (photo attached; triggers `propose_quick_calories`)
- "Save this as a food called 'homemade granola'." (photo attached; triggers `propose_food`)
- "What's my adaptive TDEE and how confident is it?"
- "How does my HRV this week compare to my baseline?"

The system prompt is long and lives in `src/components/ai/Trace.svelte` if you want to see exactly what Trace is told about which tool to use when.

## Smart Log (voice)

Smart Log is press-and-hold voice logging. Hold the Trace FAB on the Diary, speak, release. On Android the transcription uses the **system speech recognizer**; on the browser it uses the **Web Speech API**. Only the transcript reaches your configured LLM; the audio stays on-device.

The transcript is passed to Trace along with a context window that includes your saved Foods, saved Meals, saved Recipes, yesterday's diary rows, and your water containers. Trace matches against those first, splits the utterance into meal slots by keyword (Breakfast, Lunch, custom names you set), and either logs directly or proposes a card for review depending on match confidence and the tool it picked.

Toggles under **Settings → AI Assistant**:

- `quickLogEnabled`: turns Smart Log on. When off, the FAB is a chat-only entry point.
- `smartLogVoiceLang`: the recognizer's language. Auto-detects from the app locale by default.

Meal-slot detection uses your configured meal names from **Settings → Diary → Meal names**, so a custom slot like "Post-workout" is a routing keyword too.

## Scan Label (photo)

Different flow, same LLM pipe. In the food editor, the **Scan Label** button on the Nutrition card lets you photograph a nutrition-facts panel. The image goes to your configured AI provider (base64 over the AI chat endpoint, 12 MB body limit) and every field it extracts (calories, macros, micros, serving size) is filled in one tap.

Requires a vision-capable model. Claude and OpenAI's vision-tier models work; Gemini's vision models work; for OpenAI-compatible endpoints, use a multimodal model (Llama vision variants, Qwen-VL, and so on).

## Goal Insights

`aiGoalInsights` turns on a proactive analysis of intake vs targets, surfaced from the Trace pane. Trace looks at trailing averages, current goal mode, and any weight trend, and offers observations. Not scheduled reminders; on-demand analysis when you open the pane.

## History and rate limits

Chat history is stored in `ai_chat_history` and reachable via `GET / POST / DELETE /api/ai/history`. The `POST /api/ai/chat` endpoint is wrapped by `aiChatLimit` middleware to keep a single user from blowing through your provider budget with a runaway loop.

## Provider setup

Provider (Claude / OpenAI / Gemini / OpenAI-compatible), API key, model, and base URL are handled the same way as CookTrace and LiftTrace: either per-user in **Settings → AI Assistant**, or env-locked at the container. See the shared [Trace setup page](../trace/setup.md) for the full walkthrough, including the model-picker's `__custom__` slot for models not in the built-in roster and how the four provider slots differ in cost and latency.

## Related

- [Trace setup (shared)](../trace/setup.md)
- [Model reference](../trace/models.md)
- [Local LLM providers](../trace/local-llm.md)
- [Foods, meals & recipes](foods.md)
- [Diary & meal logging](diary.md)
