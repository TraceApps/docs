# NoteTrace

NoteTrace is a self-hosted alternative to Google Keep: notes, checklists, and reminders in a clean card grid, in a single Docker container on your own hardware. One Node/Express server, one Svelte PWA, a SQLite database, and a Capacitor Android app that either runs fully offline or syncs against the same server. AGPL-3.0, no telemetry, nothing leaves your network unless you point it at a push service or AI provider yourself.

The idea is Keep's speed with the pieces Keep leaves out: open it and start typing, and when you need more there is Markdown formatting, voice notes with searchable transcripts, version history, repeating reminders that keep their local time, sharing with other people on your server, and a one-click move out of Google Keep.

!!! note "Dev pre-release"
    NoteTrace is the newest Trace app. It is feature complete and in testing toward v1.0.0, with builds published on the dev channel: the `:dev` Docker image and a signed APK on the [Releases page](https://github.com/TraceApps/notetrace/releases). These pages describe the app as it stands there.

![NoteTrace notes grid with pinned notes, checklists, reminders, and labels](../assets/img/notetrace/01-notes.png)

## Compared to other note apps

NoteTrace isn't trying to be a knowledge base. It's for the notes you'd put in Keep.

- **Google Keep** is quick and simple, and it's Google's. NoteTrace keeps the card grid, checklists, colors, labels, pins, archive, and reminders, runs on your own server, and imports your Keep notes, photos included, straight from Google Takeout.
- **Memos** and **Blinko** are timeline-first and lean on tags and AI. NoteTrace is grid-first with real checklists and reminders, and imports straight from both.
- **Evernote** grew into a heavyweight workspace. NoteTrace keeps the everyday part (quick notes, checklists, clipped links, reminders) and imports your `.enex` exports with their images and tags.
- **Obsidian**, **Joplin**, and **Trilium** are built for long-form, linked documents. NoteTrace stores Markdown too, with the same `[[links]]`, so notes stay portable, but it's tuned for short notes and lists you check every day.

## What's inside

- Card grid with a pinned section, quick capture, and a layout that fills wide screens (1 to 6 columns).
- Rich editor that saves Markdown, and checklists with drag to reorder. Switch any note between text and checklist without losing content.
- Drawings: pen, marker, highlighter, eraser, and lasso on a paper or dark sheet, with stylus pressure, palm rejection, pinch zoom, and every stroke still editable later.
- Images and files on any note: add, paste, drag in, or share them from another app. PDFs open page by page with a picture of the first page on the note, and any file can be downloaded or opened in another app.
- Labels with colors and icons, nested labels, sixteen note colors, archive, and a trash that empties itself after 30 days.
- Full-text search across titles, bodies, checklist items, voice note transcripts, text in images, and the text inside PDFs, one Ctrl+K away, with matches marked on the cards and recent searches kept.
- `[[Note title]]` links with a Linked From section, and a timeline view grouped by day.
- Voice notes: record with pause and a level meter (with the screen off in the Android app), a waveform to scrub, play speed, audio files and Keep recordings, and transcripts from Trace with tappable timestamps.
- A Tasks view of dated checklist items and the lists you choose: edit, move, and delete in place, repeating tasks, a Completed section, drag to reorder, swipe on a phone, and a daily Tasks Due notification. And a List layout that opens notes beside the list.
- Trace in the editor (Tidy Up, Summarize, Make a Checklist) and a Trace chat that can find and change notes; the same note tools for external AI agents over [MCP](mcp.md).
- Your [CookTrace](cooktrace.md) shopping list in NoteTrace, grouped by aisle and usable offline, and checklists sent to it in one tap.
- Print a note or save it as a PDF, and start new notes from your own templates with the date filled in.
- Version history: every editing session leaves a restore point, sync conflicts never throw an edit away, and Undo covers deleted items, removed attachments, and text/checklist switches.
- Thousands of notes stay quick: they're drawn a screenful at a time as you scroll.
- The web app keeps working offline: it opens with the notes and files it has seen, keeps your edits, and syncs them when you're back, newer changes winning and nothing lost.
- Reminders, one-off or repeating (daily, weekly, monthly, yearly): exact alarms on the phone, notifications in the browser, and pushes through ntfy, Gotify, or Apprise.
- Sharing with view or edit access for other accounts on your server.
- Import from Google Keep (Takeout, photos included), Evernote, Memos, Blinko, and Markdown; export everything as Markdown with images.
- Android app with share-sheet capture for text, links, photos, and audio, Quick Note and Notes home screen widgets, a New Note Quick Settings tile, home screen shortcuts, an optional fingerprint or face app lock, and full offline mode.
- Layouts that follow the screen, including foldables: each screen keeps its own layout, and a half-open phone keeps notes off the crease.
- Accessible: keyboard use throughout, named controls for screen readers, AA contrast in both themes, and zoom.
- The shared Trace foundation: OIDC SSO, backups, in-app updates, Trace AI, API tokens, webhooks.

## Get started

- [Install with Docker Compose](../getting-started/compose.md) has a NoteTrace tab.
- [Feature tour](features.md) walks through the app before you install.
- [Import and export](import-export.md) covers moving your notes in from Google Keep, Evernote, and others.

## Shared setup pages

Authentication, Trace AI, backups, push notifications, reverse proxies, and the Android sync model work the same way in NoteTrace as in the other Trace apps, so those pages apply here too. Where NoteTrace differs, its own pages say so.

## Related

- [Mobile install](../mobile/install.md)
- [Push notifications overview](../integrations/notifications.md)
- [How the apps compare](../self-hosting/compare.md)
