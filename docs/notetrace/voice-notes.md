# Voice notes

A voice note is a recording attached to a note. Record one in the app, or bring in audio from a file, another app, or Google Keep. With [Trace](trace.md) set up, each one is transcribed, and the transcript is searchable and can be tapped to jump to that point in the recording.

![A voice note with its waveform and a timestamped transcript](../assets/img/notetrace/voice-note.png)

## Start a voice note {#record}

There's a quick way from wherever you are:

- **Take a note**: the microphone button on the bar starts a new voice note.
- **Phone**: tap the round **+** button and pick **Voice Note**, or hold **+** to start recording straight away.
- **List layout**: the arrow next to **New Note** offers **New Voice Note**.
- **Keyboard**: press `v` in the notes list. See [Keyboard shortcuts](shortcuts.md).
- **Home screen**: long-press the NoteTrace icon (the Android app, or the installed web app on Android) and pick **Voice Note**. **New Note** and **New List** are there too.
- **In a note**: the microphone button in the editor's toolbar (on a phone too), or type `/voice` as a [slash command](shortcuts.md#slash-commands).

A quick voice note opens straight into recording. When you stop, the transcript becomes the note's text and Trace suggests a short title. If you discard the recording and haven't written anything, the empty note closes.

## Recording {#recording}

The recorder shows the time and a live level meter, so you can see it's hearing you.

- **Pause** and **Resume** as often as you like; the length leaves the pauses out.
- **Stop and Save** attaches the recording; **Discard** throws it away.
- **Add an audio file instead** swaps the recorder for a file picker.

| Where | Longest recording | Screen off |
|---|---|---|
| Android app | 3 hours | Keeps recording. A notification shows it's recording, with **Pause** or **Resume** and **Stop**. |
| Browser and installed web app | 60 minutes | Recording can stop when the screen turns off or you switch apps. |

Recordings are mono and compact (32 kbps), so an hour is about 14 MB. The first recording asks for microphone access.

## Listening {#playback}

Each voice note shows a waveform with its length.

- **Tap or drag the waveform** to move to any point. With the keyboard, focus it and use the arrow keys (5 seconds, or 30 with Shift).
- **The speed button** steps through 1×, 1.25×, 1.5×, and 2×. The speed is remembered on each device.
- **Picking up where you left off**: a voice note you stopped partway through starts from the same spot next time, on that device.

Voice notes sync to your other devices like images do, and in Android local mode they stay on the phone until you connect to a server.

## Audio files {#files}

Audio you already have becomes a voice note too:

- **In the editor**: **Add an audio file instead** in the recorder, **Add Audio File** in a phone's **Add** sheet, or drop or paste audio files onto the note.
- **From another app**: share a recording from a voice recorder, a chat, or a file manager, and pick NoteTrace. In the Android app and in the installed web app on Android it opens a new note with the recording on it.
- **From Google Keep**: voice recordings in a Takeout export are imported with their notes. See [Import and export](import-export.md#keep).

MP3, M4A, WAV, Ogg, Opus, WebM, AAC, and FLAC play as they are. Formats browsers can't play, such as Keep's 3GP and AMR recordings, are converted to M4A by the server when you add them. That needs a connected server with audio support (the NoteTrace Docker image includes it); in Android local mode those files can't be added.

## Transcripts {#transcripts}

With **Transcribe Voice Notes** on (Settings, Trace; on by default), each new voice note is transcribed. The transcript shows under the recording and is included in search. **Add to Note** puts it into the note: a paragraph on a text note, or one item per line on a checklist. A voice note that wasn't transcribed has a **Transcribe** button.

When the provider gives times, the transcript is a list of timestamped lines. Tap a line to play from there; the line that's playing is highlighted. Long transcripts show a few lines at a time, with **Show all** to see the rest.

| Provider | Transcription | Timestamps |
|---|---|---|
| OpenAI | `gpt-4o-mini-transcribe` by default | With `whisper-1` set as the **Transcription Model** (the `gpt-4o` models return text only) |
| Gemini | Your chosen Gemini model | Yes |
| OpenAI-compatible (Speaches, LocalAI, and other Whisper servers) | `whisper-1` by default | Yes, when the server supports `verbose_json` |
| Claude | No; Claude can't transcribe audio | |

### Long recordings

Transcription services limit how much audio one request can carry (25 MB for OpenAI, about 13 MB for Gemini). A longer recording is split into pieces, each piece is transcribed, and the transcripts are joined with their timestamps lined up. The server does the splitting when you're connected to one; the Android app splits its own recordings on the phone in local mode.

When Trace is set by environment variables, the server transcribes and splits voice notes itself. See [Where the requests go](trace.md#privacy).

## How it's stored {#storage}

A voice note is an audio attachment on the note, with its length, a small waveform, its transcript, and the transcript's timestamps. See [Images and voice notes](notes.md#attachments).

## Related

- [Trace in NoteTrace](trace.md)
- [Android](android.md)
- [Settings reference](settings.md#trace-ai-per-user)
