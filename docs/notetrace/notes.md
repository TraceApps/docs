# Notes, checklists, and labels

How notes are stored and behave, for when the [feature tour](features.md) isn't enough detail.

## Text notes

The body is stored as Markdown in `notes.body_md`. The editor is [TipTap](https://tiptap.dev/) with its Markdown extension, so what you see as bold, lists, headings, quotes, and code is saved as `**bold**`, `- item`, `# Heading`, `> quote`, and backticks. A single line break inside a paragraph is saved as a Markdown hard break (two trailing spaces), so notes pasted from plain text keep their lines.

Titles are separate from the body and optional. An untitled note shows its first lines on the card.

## Checklists

Each checklist item is its own row with a stable id (a UUID generated on the device that created it), its text, checked state, and position. Items are never replaced as a whole list: adding, editing, checking, reordering, or deleting touches only that item, and a delete leaves a tombstone. That's what lets two phones edit the same list offline and still merge cleanly (see [How sync works](../mobile/sync.md)).

Switching a checklist to text writes one line per item, with checked items wrapped in `~~strikethrough~~`, separated by blank lines. Switching text to a checklist turns each non-empty line into an item, strips bullets and numbering, and treats `- [x]` tasks and struck-through lines as checked.

When every item on a checklist is checked, the `checklist.completed` [webhook](webhooks.md) fires.

## Links between notes {#links}

Type `[[` in a text note to pick another note by title, or type the whole `[[Note title]]`; it becomes a link chip. Tap the chip to open that note. If no note has that title yet, NoteTrace offers to create it. In the Markdown body the link is plain `[[Note title]]`, the same syntax Obsidian and other Markdown apps use, so it survives export and import.

A note that other notes link to shows a **Linked From** section at the bottom of the editor, listing those notes. Titles match ignoring case.

When you rename a note, links to it in your own notes update to the new title once you close the editor, so a half-typed title never touches your links. Links in notes that other people own and share with you aren't changed, and nothing is rewritten when another note already has the old or the new title, since those links could belong to it.

## Images and voice notes

Images are stored per note in `note_attachments`, each with a stable id, like checklist items, so adding or removing an image on one device merges with changes elsewhere. The file itself is uploaded to the server's uploads folder first (scaled down in the browser when it's a large photo); a note links to files on its own server only. In Android local mode photos are saved on the phone and uploaded by the first sync after you connect. A note holds up to 50 images.

Voice notes are audio attachments in the same table, with their length. A transcript of a voice note, or the text [Trace read from an image](trace.md#image-text), is saved on that attachment and included in search. See [Trace in NoteTrace](trace.md).

## Pin, archive, trash

- **Pin** keeps a note in the pinned section. Archiving or trashing a note unpins it.
- **Archive** moves a note out of the grid and into the Archive view, where it can still be searched.
- **Trash** keeps a note for 30 days. A server task checks every 15 minutes and permanently deletes notes trashed longer than that; in Android local mode the app does the same when it opens. Permanent deletes sync to every device.

## Labels

Labels belong to the account that made them, have an optional color, and can be reordered in **Edit Labels**. Label names are unique per account, ignoring case. Deleting a label removes it from its notes but doesn't delete the notes.

On a [shared note](sharing.md), labels are personal: each person sees only the labels they added.

## Colors

Notes can be plum, tide, moss, sand, clay, rose, or the default. Each color has a dark and a light variant, so a colored card stays readable in either theme.

## Search

**Ctrl+K** (**Cmd+K**) focuses search from anywhere in the app. Search uses SQLite FTS5 over titles, bodies, checklist item text, voice note transcripts, and text read from images, kept current by database triggers. Each word you type matches as a prefix, and all words must match. Results are limited to the view you search from (Notes, Archive, Trash, or a label).

## Timeline view {#timeline}

The **Timeline View** button beside the search box switches between the card grid and a timeline: one column of notes grouped by the day each was last edited (Today, Yesterday, the weekday for the past week, then dates). Pinned notes stay at the top. The choice is remembered on each device and applies to Notes, Archive, Trash, and labels.

## Version history

Snapshots are stored on the server in `note_versions`:

| Reason | When |
|---|---|
| Edit | The first change after 10 minutes without a snapshot, so one editing session is one restore point |
| Restore | Before restoring an older version, and before switching between text and checklist |
| Conflict | When a device pushes an edit older than the server's copy; the older edit is saved here instead of being lost |

The newest 50 snapshots per note are kept. Android local mode keeps its own history on the device.

## Related

- [Reminders](reminders.md)
- [Sharing](sharing.md)
- [How sync works](../mobile/sync.md)
