# Notes, checklists, and labels

How notes are stored and behave, for when the [feature tour](features.md) isn't enough detail.

## Text notes

The body is stored as Markdown in `notes.body_md`. The editor is [TipTap](https://tiptap.dev/) with its Markdown extension, so what you see as bold, lists, headings, quotes, and code is saved as `**bold**`, `- item`, `# Heading`, `> quote`, and backticks. A single line break inside a paragraph is saved as a Markdown hard break (two trailing spaces), so notes pasted from plain text keep their lines.

Titles are separate from the body and optional. An untitled note shows its first lines on the card.

## Checklists

Each checklist item is its own row with a stable id (a UUID generated on the device that created it), its text, checked state, and position. Items are never replaced as a whole list: adding, editing, checking, reordering, or deleting touches only that item, and a delete leaves a tombstone. That's what lets two phones edit the same list offline and still merge cleanly (see [How sync works](../mobile/sync.md)).

Switching a checklist to text writes one line per item, with checked items wrapped in `~~strikethrough~~`, separated by blank lines. Switching text to a checklist turns each non-empty line into an item, strips bullets and numbering, and treats `- [x]` tasks and struck-through lines as checked.

When every item on a checklist is checked, the `checklist.completed` [webhook](webhooks.md) fires.

## Due dates and Tasks {#due}

A checklist item can have a due date: a calendar day, stored as `due_date` on the item and synced with it, so there's no time zone to get wrong. Set it from the item's calendar button in the editor or from the Tasks view. The Tasks view lists unchecked items that have a due date plus every unchecked item on checklists set to Show in Tasks (your own and ones shared with you), grouped by due date or by list; checking one there checks it in its note. Show in Tasks is stored as `notes.in_tasks` for a note's owner and on the membership for someone it's shared with, so each person chooses for themselves; a checklist named Tasks starts out shown. Trace and MCP see an item's due date in `get_note`, list tasks with `list_tasks` (`all_checklists` for every open item), turn Show in Tasks on or off with `show_in_tasks` on `create_note` and `update_note`, and set dates with `set_due_date` or the `due` argument of `add_checklist_items`.

**Tasks Due** (in Settings, Notifications, off by default) sends one notification a day at the time you pick (9:00 by default) listing items due today and overdue, and opens the Tasks view when tapped. It uses the same delivery as reminders: on Android the phone schedules it, in a browser it fires while NoteTrace is open, and with a push service the server sends it once per day in your time zone (catching up until 20:00 if the server was down at that time). Nothing is sent on a day with nothing due.

## Links between notes {#links}

Type `[[` in a text note to pick another note by title, or type the whole `[[Note title]]`; it becomes a link chip. Tap the chip to open that note. If no note has that title yet, NoteTrace offers to create it. In the Markdown body the link is plain `[[Note title]]`, the same syntax Obsidian and other Markdown apps use, so it survives export and import.

A note that other notes link to shows a **Linked From** section at the bottom of the editor, listing those notes. Titles match ignoring case.

When you rename a note, links to it in your own notes update to the new title once you close the editor, so a half-typed title never touches your links. Links in notes that other people own and share with you aren't changed, and nothing is rewritten when another note already has the old or the new title, since those links could belong to it.

## Images and voice notes {#attachments}

Images are stored per note in `note_attachments`, each with a stable id, like checklist items, so adding or removing an image on one device merges with changes elsewhere. The file itself is uploaded to the server's uploads folder first (scaled down in the browser when it's a large photo); a note links to files on its own server only. In Android local mode photos are saved on the phone and uploaded by the first sync after you connect. A note holds up to 50 images.

Voice notes are audio attachments in the same table, with their length (`duration_ms`), a waveform of 64 bar heights (`waveform`), and, once transcribed, the transcript's timestamped lines (`segments`, seconds from the start). A transcript of a voice note, or the text [Trace read from an image](trace.md#image-text), is saved on that attachment as `extracted_text` and included in search. All of it syncs with the attachment. Audio a browser can't play (Google Keep's 3GP and AMR) is converted to M4A by the server's built-in ffmpeg on upload. See [Voice notes](voice-notes.md).

## Pin, archive, trash

- **Pin** keeps a note in the pinned section. Archiving or trashing a note unpins it.
- **Archive** moves a note out of the grid and into the Archive view, where it can still be searched.
- **Trash** keeps a note for 30 days. A server task checks every 15 minutes and permanently deletes notes trashed longer than that; in Android local mode the app does the same when it opens. Permanent deletes sync to every device.

## Labels

Labels belong to the account that made them, have an optional color and icon, and can be reordered in **Edit Labels**. Tap the dot beside a label there to pick its color and one of 48 icons; the icon replaces the dot in the sidebar, the icon rail, and the label picker, and syncs with the label. Label names are unique per account, ignoring case. Deleting a label removes it from its notes but doesn't delete the notes.

On a [shared note](sharing.md), labels are personal: each person sees only the labels they added.

A slash in a name nests a label: `Home/Garage` and `Home/Kitchen` show under `Home` in the sidebar and in the label picker, each group folds, and opening `Home` shows notes from `Home` and every label under it. Nesting is only in the name, so it survives export, import, and sync unchanged.

## Order and density {#order}

Notes are newest edit first until you drag one; then Notes, Shared with Me, and label views use **Your Order**, saved as a setting so every device shows the same arrangement. Reordering doesn't change a note's edited time or add to its version history. New notes appear at the top. **Settings, Appearance, Note Order** switches between the two.

**Card Density** in the same place makes cards compact: smaller text, fewer checklist lines, and link previews without their image.

## Link previews {#link-previews}

When a note contains a link, the card shows a preview of the first one: the page title, its image, and the site. Your NoteTrace server fetches the page (not your browser or phone), through the same address checks webhooks use, and keeps the result for a week. Links to private or loopback addresses get no preview unless the server has `ALLOW_PRIVATE_LINK_PREVIEWS=1`. Turn previews off in **Settings, Appearance, Link Previews**. They need a server, so Android local mode doesn't show them.

## Colors

Notes can be one of sixteen colors (ember, clay, amber, sand, lime, moss, sage, mint, sky, tide, indigo, plum, orchid, rose, bark, slate) or the default. Each color has a dark and a light variant, so a colored card stays readable in either theme.

## Search

**Ctrl+K** (**Cmd+K**) focuses search from anywhere in the app. Search uses SQLite FTS5 over titles, bodies, checklist item text, voice note transcripts, and text read from images, kept current by database triggers. Each word you type matches as a prefix, and all words must match. Results are limited to the view you search from (Notes, Archive, Trash, or a label).

## Timeline view {#timeline}

**Timeline** in **View Options** (in the page header) switches the notes to a timeline: one column of notes grouped by the day each was last edited (Today, Yesterday, the weekday for the past week, then dates). Pinned notes stay at the top. The choice is remembered on each device and applies to Notes, Archive, Trash, and labels.

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
