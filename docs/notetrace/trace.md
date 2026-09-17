# Trace in NoteTrace

Trace is the AI assistant shared by the Trace apps. In NoteTrace it works on your notes: it can clean one up, find and change notes from a chat, write out voice notes, and read the text in images. Provider, key, and model setup is the same as in every Trace app: see [Setting up Trace](../trace/setup.md), [Cloud providers](../trace/cloud.md), and [Local LLMs](../trace/local-llm.md).

Nothing on this page runs until Trace is set up, and each feature sends only what it needs to your AI provider.

## Ask Trace in the editor {#editor}

On a text note, the **Ask Trace** button (sparkle icon) in the editor bar offers three actions:

| Action | What it does | Apply with |
|---|---|---|
| **Tidy Up** | Fixes spelling and grammar and adds structure, keeping your meaning and facts | **Use This** replaces the text |
| **Summarize** | A few bullet points with the key facts and to-dos | **Add to Top** puts the summary above the text |
| **Make a Checklist** | Turns the note into checkable items | **Make Checklist** converts the note |

Trace's result is shown first, and nothing changes until you apply it. Applying always saves the text you had as a restore point in [version history](notes.md#version-history), even if you edited a minute ago, so an unwanted rewrite is one tap to undo. **Try Again** asks for a fresh result.

## Chat with your notes {#chat}

Open Trace from the round button and ask in plain words. Trace can call note tools, so it answers from your real notes and can change them:

- "What's on my groceries list?"
- "Start a packing list for the cabin: charger, boots, headlamp."
- "Add oat milk to groceries and check off the coffee."
- "Remind me about the car registration next Friday at 9."
- "What's due this week?" or "Make the taxes item due April 15."
- "Label everything about the homelab as Homelab."
- "Move the old moving checklist to the trash."

Reminder times are read in your own time zone (the one your device is set to); Trace is told the current date and time with every message. Trace follows the same rules you do: it can't edit a note shared with you as view-only, and only a note's owner can set its reminder or move it to the trash. When Trace rewrites a note's text, the previous text goes to version history. Trash is always restorable for 30 days.

The full tool list is in the [Trace tool catalog](../reference/trace-tools.md#notetrace). The same tools are available to external AI agents over [MCP](mcp.md).

## Voice notes {#voice}

With **Transcribe Voice Notes** on (Settings, Trace; on by default), Trace writes out each new voice note. The transcript shows under the recording with timestamps you can tap to jump to, is included in search, and can be added to the note with **Add to Note**: as a paragraph on a text note, or as items on a checklist. A voice note that wasn't transcribed has a **Transcribe** button. A quick voice note (the microphone on Capture a thought) also gets its text from the transcript and a title suggested by Trace.

Transcription needs a provider that accepts audio:

| Provider | Transcription | Timestamps |
|---|---|---|
| OpenAI | Yes (`gpt-4o-mini-transcribe` by default) | With `whisper-1` as the Transcription Model |
| Gemini | Yes (your chosen Gemini model) | Yes |
| OpenAI-compatible | Yes, when the server has a Whisper-style `/v1/audio/transcriptions` endpoint (`whisper-1` by default), such as Speaches or LocalAI | When the server supports `verbose_json` |
| Claude | No; Claude can't transcribe audio | |

**Transcription Model** in Settings, Trace overrides the default model name. When Trace is set by environment variables, the server does the transcription and `AI_TRANSCRIBE_MODEL` sets the model. Recordings longer than a provider takes in one request are split, transcribed in pieces, and joined with their timestamps lined up.

Recording, playback, audio files, and imports are covered in [Voice notes](voice-notes.md).

## Text in images {#image-text}

Open an image full screen and tap **Read Text**. Trace reads the text in it (a receipt, a whiteboard, a screenshot, a page of a book), shows it, and saves it with the image so search finds the note by that text. **Add to Note** copies it into the note, and **Copy** puts it on the clipboard.

Turn on **Read Text in New Images** (Settings, Trace; off by default) to have Trace read every image you add. This sends each image to your AI provider, which is why it's off until you choose it.

Reading images works with Claude, OpenAI, Gemini, and OpenAI-compatible servers running a vision model. Images are scaled down to 1600px on the longest side before they're sent.

## Where the requests go {#privacy}

With your own key, Trace calls your provider directly from your browser or phone. When an admin sets Trace with environment variables (`AI_PROVIDER`, `AI_API_KEY`), requests go through the NoteTrace server instead, so the key never reaches the device. The chat still works with your notes in that setup: the server passes each request on to the provider with its key and the configured model, and the note tools run on your device as usual. Transcripts and image text are saved on your NoteTrace server with the note.

With Trace set by environment variables, voice note transcription and image reading follow the server's provider: on a Claude server there's no transcription, so voice notes are saved without a transcript and no Transcribe button shows.

## Related

- [Setting up Trace](../trace/setup.md)
- [Model Context Protocol (MCP)](mcp.md)
- [Notes, checklists, and labels](notes.md)
- [Voice notes](voice-notes.md)
