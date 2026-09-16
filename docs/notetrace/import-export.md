# Import and export

Everything lives under **Settings, Import & Export**. Imports and exports work the same on the web, on Android connected to a server, and in Android local mode.

![Import and Export settings](../assets/img/notetrace/05-import-export.png)

Importing the same export twice is safe: notes that are already there (same title, text, type, and created date) are skipped, and their images aren't uploaded again. After each import, a summary lists what came across and what didn't.

## Google Keep {#keep}

1. Go to [Google Takeout](https://takeout.google.com/), click **Deselect all**, then select only **Keep**.
2. Create the export and download the `.zip` when Google emails you.
3. In NoteTrace, open **Settings, Import & Export** and choose **Google Keep, Choose File**. Pick the zip (no need to unzip it).

What comes across:

| Google Keep | NoteTrace |
|---|---|
| Title and text | Title and body, with line breaks kept |
| Checkbox lists | Checklist with checked state and order |
| Photos and drawings | Images on the note |
| Labels | Labels, matched to existing ones by name or created |
| Colors | Nearest NoteTrace color (see below) |
| Pinned, archived | Pinned, archived |
| Links (web link previews) | Added to the end of the note |
| Created and edited dates | Kept |
| Trash | Left out, unless **Include notes from Keep's trash** is on |
| Voice recordings | Voice notes on the note. Keep's 3GP/AMR files are converted to M4A by the server; in Android local mode, or on a server without audio support, the summary counts them as not added |
| Other attachments | Files on the note, with a PDF's first page and text read for search |
| Reminders, collaborators | Not in Takeout's note files, so not imported |

Color mapping: red becomes ember, orange becomes amber, yellow becomes sand, green becomes moss, teal becomes sage, blue becomes sky, dark blue becomes tide, purple becomes plum, pink becomes rose, brown becomes bark, gray becomes slate, and default stays uncolored.

!!! tip "Big exports"
    The zip is read on your device or in your browser. Notes are saved first, then their images are uploaded, with a progress count for each. The server accepts about 60 image uploads a minute, so a Keep export with hundreds of photos pauses now and then to stay under that limit; leave the page open until the summary appears.

## Evernote {#evernote}

1. In Evernote, export a notebook (or selected notes) and choose the **ENEX** format. Evernote writes one `.enex` file per export.
2. In NoteTrace, choose **Evernote, Choose File** and pick the `.enex` file. To bring several notebooks over at once, zip their `.enex` files together and pick the zip.

What comes across:

| Evernote | NoteTrace |
|---|---|
| Title and text | Title and body, one line per Evernote line, with bold, italic, strikethrough, links, headings, lists, quotes, and code |
| Checkboxes (older and Evernote 10 style) | A checklist when the note is only checkboxes; otherwise `- [ ]` lines in the text, which **Show Checkboxes** turns into a checklist |
| Evernote 10 tasks | Checklist items |
| Images | Images on the note, in the order the note shows them |
| Tables | One line per row, with cells separated by a vertical bar |
| Tags | Labels |
| Notebook | A label with the notebook's name (the export's file name), when **Label notes with their notebook name** is on |
| Created and updated dates | Kept |
| Source URL (web clips) | Added to the end of the note |
| Reminders | Kept when they're still ahead and not marked done |
| PDFs, audio, other files | Files on the note, with a PDF's first page and text read for search |
| Encrypted text | Replaced with "(encrypted text not imported)" |

The whole `.enex` file is read on your device, so a very large export can be slow on a phone. Exporting one notebook at a time keeps each file manageable.

## Memos {#memos}

Memos has no export file, so NoteTrace reads your memos straight from your Memos server.

1. In Memos' settings, create a personal access token.
2. In NoteTrace, under **Memos**, enter your Memos address (for example `https://memos.example.com`) and paste the token.
3. Tap **Import**.

Your memos' content, tags (as labels), images and other attached files, pinned state, archived memos, and created and updated times come across. Only your own memos are imported, not other people's public memos, and comments aren't. The token is used once from your browser or phone and isn't saved; you can delete it in Memos afterward.

!!! note "https"
    If NoteTrace is on `https`, your browser only lets it reach a Memos server that's also on `https`. An address typed without `https://` tries `https` first, then `http`.

## Blinko {#blinko}

Current Blinko versions no longer create backups from their settings (Blinko has that option turned off), so use Blinko's **Markdown** export: in Blinko, open **Settings, Export**, set **Export Format** to Markdown, and export. In NoteTrace, import the zip with **Markdown Files, Choose File** (below). Note text, `#tags` (as labels), images, and creation dates come across. Pinned and archived status don't, because Blinko's export leaves them out, and the export includes notes in Blinko's trash. Blinko's JSON and CSV exports carry only the text and date, without images, so NoteTrace doesn't read them.

If you have a `.bko` backup from an older Blinko version, choose **Blinko Backup, Choose File** and pick it.

Notes, images and other attached files, tags (Blinko keeps them in the text, so they become labels), pinned, archived, and dates come across. A Blinko backup holds every account on that Blinko server; NoteTrace imports the account whose name matches your NoteTrace username, or the only account when there's just one. Blinko's backup doesn't mark which notes are in its trash, so those come across as regular notes.

## Markdown files {#markdown}

Choose **Markdown Files, Choose File** and pick a `.zip` of `.md`, `.markdown`, or `.txt` files, or a single file. This covers Obsidian vaults, Joplin Markdown exports, Blinko's Markdown export, a NoteTrace export, and any other app that exports Markdown.

How each file becomes a note:

- **Title**: `title` in front matter, else a leading `# Heading` (removed from the body), else the file name. File names that are just dates, ids, `note-<id>-<time>` (Blinko), or "Untitled" leave the note untitled.
- **Body**: the Markdown after the front matter. `.txt` files keep their line breaks.
- **Images**: images the note embeds from inside the zip, `![](images/photo.jpg)` or Obsidian's `![[photo.jpg]]`, become images on the note. Images on the web (`https://...`) stay in the text as they are.
- **Checklist**: a file whose lines are all tasks (`- [ ]` and `- [x]`) becomes a checklist, as does `kind: checklist` in front matter.
- **Labels**: `labels` or `tags` in front matter, plus inline `#tags` in the text when **Turn #tags into labels** is on. Headings, code, and `#123` numbers aren't treated as tags.
- **Dates**: `created` (or `created_at`, `date`) and `updated` (or `updated_at`, `modified`). Blinko file names carry the creation time.
- **Other front matter**: `color`, `pinned`, `archived`, `reminder`, `repeat`, and `timezone`, as written by a NoteTrace export.
- Files inside a folder named `Archive` are archived.
- Hidden folders and files such as `.obsidian`, `.trash`, and `__MACOSX` are skipped.

## Export as Markdown {#export}

**Export as Markdown** builds a zip of every note you can see, including notes shared with you, with their images, voice notes, and files:

```
NoteTrace/
  Notes/
    Groceries.md
    Homelab to-do.md
  Archive/
    Old project.md
  attachments/
    5f0c...e1.jpg
```

Trashed notes aren't exported. Each file has front matter with everything needed to import it back, and its images as embeds at the top, so any Markdown app shows them. Voice notes and other files are links at the end, each under the name it was added with:

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

![](../attachments/5f0c...e1.jpg)

- [ ] Chicken thighs
- [x] Coffee beans
```

Notes shared with you also carry `shared_by`. On Android the zip opens the share sheet so you can save it to Files or send it somewhere; in a browser it downloads.

The Markdown export is for portability. For a complete copy of your server, including accounts, settings, sharing, and version history, use a [full backup](../self-hosting/backups.md).
