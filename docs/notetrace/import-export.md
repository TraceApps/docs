# Import and export

Everything lives under **Settings, Import & Export**. Imports and exports work the same on the web, on Android connected to a server, and in Android local mode.

![Import and Export settings](../assets/img/notetrace/05-import-export.png)

## Import from Google Keep {#keep}

1. Go to [Google Takeout](https://takeout.google.com/), click **Deselect all**, then select only **Keep**.
2. Create the export and download the `.zip` when Google emails you.
3. In NoteTrace, open **Settings, Import & Export** and choose **Google Keep, Choose File**. Pick the zip (no need to unzip it).

What comes across:

| Google Keep | NoteTrace |
|---|---|
| Title and text | Title and body, with line breaks kept |
| Checkbox lists | Checklist with checked state and order |
| Labels | Labels, matched to existing ones by name or created |
| Colors | Nearest NoteTrace color (see below) |
| Pinned, archived | Pinned, archived |
| Links (web link previews) | Added to the end of the note |
| Created and edited dates | Kept |
| Trash | Left out, unless **Include notes from Keep's trash** is on |
| Images, drawings, audio | Not imported yet; the summary counts them |
| Reminders, collaborators | Not in Takeout's note files, so not imported |

Color mapping: red and orange become clay, yellow and brown become sand, green becomes moss, teal, blue, and dark blue become tide, purple becomes plum, pink becomes rose, and gray and default stay uncolored.

!!! tip "Big exports"
    The zip is read on your device or in your browser, and only the note text is sent to the server, so a large Keep export full of photos still imports quickly. Importing the same export twice is safe: notes that are already there are skipped.

## Import Markdown files {#markdown}

Choose **Markdown Files, Choose File** and pick a `.zip` of `.md`, `.markdown`, or `.txt` files, or a single file. This covers Obsidian vaults, Memos and Joplin Markdown exports, a NoteTrace export, and any other app that exports Markdown.

How each file becomes a note:

- **Title**: `title` in front matter, else a leading `# Heading` (removed from the body), else the file name. File names that are just dates, ids, or "Untitled" leave the note untitled.
- **Body**: the Markdown after the front matter. `.txt` files keep their line breaks.
- **Checklist**: a file whose lines are all tasks (`- [ ]` and `- [x]`) becomes a checklist, as does `kind: checklist` in front matter.
- **Labels**: `labels` or `tags` in front matter, plus inline `#tags` in the text when **Turn #tags into labels** is on. Headings, code, and `#123` numbers aren't treated as tags.
- **Dates**: `created` (or `created_at`, `date`) and `updated` (or `updated_at`, `modified`).
- **Other front matter**: `color`, `pinned`, `archived`, `reminder`, `repeat`, and `timezone`, as written by a NoteTrace export.
- Files inside a folder named `Archive` are archived.
- Hidden folders and files such as `.obsidian`, `.trash`, and `__MACOSX` are skipped.

Notes already present (same title, body, type, and created date) are skipped.

## Export as Markdown {#export}

**Export as Markdown** builds a zip of every note you can see, including notes shared with you:

```
NoteTrace/
  Notes/
    Groceries.md
    Homelab to-do.md
  Archive/
    Old project.md
```

Trashed notes aren't exported. Each file has front matter with everything needed to import it back:

```markdown
---
title: "Groceries"
kind: "checklist"
labels: ["Home"]
color: "moss"
pinned: true
reminder: "2026-09-14T13:00:00Z"
repeat: "weekly"
timezone: "America/New_York"
created: "2026-09-01T12:00:00Z"
updated: "2026-09-12T14:30:00Z"
---

- [ ] Chicken thighs
- [x] Coffee beans
```

Notes shared with you also carry `shared_by`. On Android the zip opens the share sheet so you can save it to Files or send it somewhere; in a browser it downloads.

The Markdown export is for portability. For a complete copy of your server, including accounts, settings, and version history, use a [full backup](../self-hosting/backups.md).
