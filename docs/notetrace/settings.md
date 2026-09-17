# Settings reference

Every section on the NoteTrace Settings page, and where to read more. Per-user sections apply to the signed-in account. Admin sections apply to the whole server and only appear for the `admin` role. Use the search box at the top of Settings to jump to a section by name or keyword (search for "keep", "lock", "shopping", "shortcuts", "density", or "ntfy", for example).

Settings is grouped into **Display**, **Integrations**, **App**, and **Admin**. On a wide screen the sections sit in a rail on the left; on a phone each opens as its own page.

## Profile (per-user)

Top of the page: avatar, name, and role. Opens the profile page for display name, password, biometric sign-in (Android), and sign out.

## Display

### Appearance (per-user)

- **Theme**: System Default, Dark, or Light.
- **Accent Color**: preset accents, Lavender by default.
- **Navigation Style**: **Auto** (the default) fits the screen: tab bar and ☰ menu on a phone, icons beside the page on an unfolded foldable or small tablet, the full sidebar on a larger screen. Or pick Bottom Tab Bar, Side Panel, or Both.
- **Persistent Sidebar**: on wider screens the sidebar stays open (on by default; from 600px with Auto, 768px otherwise). With Auto, an unfolded foldable or small tablet remembers its own choice of icons or the full sidebar. With Navigation Style set to Both, the tab bar sits beside it. Collapse it to icons with the button beside the logo.
- **Start Page**: open NoteTrace to Notes (the default), Tasks, Reminders, or Archive.
- **Reduce Motion**, **Page Banners**, and **Animation Style**, shared with the other Trace apps.

The pinned sidebar collapses to icons with the button beside the logo, and the ☰ at the top of the icons opens it again.

### Notes

How notes behave, rather than how the app looks.

- **Card Density**: Comfortable or Compact. Per device.
- **Show Every Checklist in Tasks**: Tasks lists every open item from every checklist, not just dated items and checklists set to Show in Tasks. Off by default; synced to your account.
- **Note Order**: Last Edited or Your Order (the order you drag notes into). Synced to your account.
- **Swipe to Archive** (touch screens): swipe a card sideways to archive it, with Undo. Per device.
- **Keyboard Shortcuts** (devices with a keyboard): single-key shortcuts on or off, and **View** to see them all. Per device. See [Keyboard shortcuts](shortcuts.md).
- **Link Previews**: show the first link's title, image, and site on cards. Synced to your account.
- **Templates**: the notes you saved to start from, with Rename and Delete (with Undo). Synced to your account. See [Templates](features.md#templates).

### Regional (per-user)

- **Date Format**: `YYYY-MM-DD`, `MM/DD/YYYY`, `DD/MM/YYYY`, or natural.
- **Time Format**: 12-hour or 24-hour. Also used by reminder times.

## Integrations

### Trace AI (per-user)

Provider, key, and model for the Trace assistant. See [Setting up Trace](../trace/setup.md).

- **Transcribe Voice Notes**: Trace writes out new voice notes (on by default). Needs OpenAI, Gemini, or an OpenAI-compatible Whisper server.
- **Summarize Long Recordings**: a recording over 10 minutes also gets a short summary when it is transcribed (off by default). Shorter recordings have a **Summarize** button on them instead. See [Voice notes](voice-notes.md#summaries).
- **Transcription Model**: overrides the default speech-to-text model name. Empty uses the default. With OpenAI, `whisper-1` gives transcripts with timestamps. See [Voice notes](voice-notes.md#transcripts).
- **Read Text in New Images**: Trace reads the text in every image you add (off by default; sends each image to your provider).

See [Trace in NoteTrace](trace.md).

### CookTrace (per-user)

Link a CookTrace server with its address and an API token with the **shopping** scope, or unlink it. Linked, **Shopping** appears in the menu. Needs a NoteTrace server, so it's unavailable in Android local mode. See [CookTrace shopping list](cooktrace.md).

## App

### Server Connection (Android)

Switch between local mode and a server, and see sync status. See [Local vs server-connected mode](../mobile/modes.md).

### App Lock (Android, per-device)

**Lock NoteTrace** and **Lock After**. See [App Lock](android.md#app-lock).

### Notifications (per-user)

- **Enable on This Device**: reminder notifications on this phone, or in this browser while NoteTrace is open. Set per device.
- **Browser Permission** (web): asks the browser once to allow notifications.
- **Exact Reminder Times** (Android): appears only when the phone isn't letting NoteTrace fire at the exact minute, with an **Allow** button.
- **Push Service**: None, Apprise, Gotify, or ntfy, with a **Send Test** button. See [Push notifications overview](../integrations/notifications.md).
- **Note Reminders**: when on (the default), the server sends due reminders through the push service. See [Reminders](reminders.md#from-the-server).
- **Tasks Due** and **Time**: one notification a day listing checklist items due today and overdue. Off by default; syncs across devices. See [Due dates and Tasks](notes.md#due).

### Import & Export (per-user)

Google Keep, Evernote, Memos, Blinko, and Markdown import, and Markdown export. See [Import and export](import-export.md).

### Backup

Full backups, scheduled auto-backups (admin), restore from a zip, and the portable JSON export. On Android local mode, a local backup zip. See [Backups and restore](../self-hosting/backups.md).

### Updates

Stable or Dev channel and the in-app updater. See [Release channels](../reference/release-channels.md).

### Diagnostics

Server and sync status, for troubleshooting.

## Admin

### Users

Invite people, change roles, reset passwords, and remove accounts. Accounts are also who you can [share notes](sharing.md) with. See [Local users, invites, roles](../auth/local-users.md).

### Authentication

Password policy, session length, and OIDC providers. See [OIDC / SSO overview](../auth/oidc.md).

### API Tokens

Personal access tokens (prefix `note_pat_`) for scripts and the MCP endpoint. See [Model Context Protocol (MCP)](mcp.md).

### Webhooks

Signed outgoing webhooks for `note.created`, `checklist.completed`, and `reminder.fired`. Needs `WEBHOOKS_ENABLED=1`. See [Webhooks](webhooks.md).

### Email

SMTP for password resets and invites. See [Email / SMTP](../integrations/smtp.md).

## About

Version, links, and licenses.
