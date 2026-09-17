# Android: share sheet, voice notes, and App Lock

The NoteTrace Android app is the same app as the web version, plus a few things only a phone can do. Install and mode selection work like the other Trace apps: see [Install the Android app](../mobile/install.md) and [Local vs server-connected mode](../mobile/modes.md).

## Save from any app

Share text, a link, photos, or any file from any Android app (a browser, a chat, the gallery, a file manager, your email) and pick **NoteTrace** in the share sheet. NoteTrace opens a new note with the shared content in it: text goes in the body, a shared page title becomes the note's title, photos are added as images, audio (a recording from a voice recorder app, say) becomes a [voice note](voice-notes.md), and anything else, like a PDF or a document, is attached as a [file](features.md#files). Up to 20 at a time. It saves on its own; close it or keep typing.

The installed web app (PWA) accepts shared text, links, and files the same way on platforms that support the Web Share Target API, such as Chrome on Android.

## Home screen shortcuts

Long-press the NoteTrace icon for **Voice Note**, **New Note**, and **New List**. Each opens the app straight into a new note; Voice Note starts recording. The installed web app offers the same shortcuts on Android. For more ways in from the home screen, see [Widgets](#widgets) and the [Quick Settings tile](#tile).

## Widgets {#widgets}

Long-press an empty spot on your home screen, choose **Widgets**, and find NoteTrace. There are two:

- **Quick Note** (4 by 1): a **Take a note** bar with buttons for a list, a voice note, and a drawing. Each opens the app straight into a new one, and the voice note starts recording.
- **Notes** (3 by 3, resizable): your pinned notes, then the ones you edited last, up to 25. Each shows its title, a few lines of text or the open items of a list, and a dot in the note's colour. Tap one to open it, **+** for a new note, or the microphone for a voice note.

The Notes widget updates whenever notes change in the app, after a sync, and when you leave the app. It's filled by the app, so a new widget says to open NoteTrace once. With **App Lock** on, it shows no notes at all, only a line saying App Lock is on. Signing out clears it.

If you force stop NoteTrace in Android's settings, Android cancels the widget's buttons; remove the widget and add it again if tapping it stops doing anything.

## Quick Settings tile {#tile}

Swipe down twice, tap the pencil to edit your Quick Settings, and drag **New Note** into place. Tapping it opens a new note from anywhere, over any app; if the phone is locked, it asks you to unlock first.

## Voice notes

The Android app records voice notes itself rather than through the web view, so recording keeps going with the screen off or another app open, for up to 3 hours. While it records, a notification shows the time with **Pause** or **Resume** and **Stop**; stop it there and the recording is saved to the note when you come back. A recording too long for your transcription service is split on the phone in local mode. See [Voice notes](voice-notes.md).

## Drawings

Drawings work with a finger or a stylus. With a stylus, fingers stop drawing (so a resting palm doesn't leave marks) and one finger moves the sheet; two fingers pan and zoom either way. The back gesture saves the drawing and closes it rather than leaving the note. See [Drawings](features.md#drawings).

## Files

Tap a file on a note to open it in NoteTrace: PDFs page by page, text as text, videos in a player. **Open With** hands any file to another app on the phone (a PDF reader, Sheets, a zip tool); if nothing on the phone opens that kind of file, the share sheet appears so you can save it to Files or Drive.

## Print and PDF

**Print or Save as PDF** in a note's **⋮** menu opens Android's print screen with the note laid out as a page. Pick a printer, or **Save as PDF** to keep a PDF in Files or Drive. It works in local mode too, pictures included. See [Print or save as PDF](features.md#print).

## Photos

Tap the image button in the editor's bottom bar to add photos from the gallery or the camera. Large photos are scaled down on the phone before they're saved, so they sync quickly. In local mode, photos stay on the phone; once you connect to a server, the next sync uploads them.

## Reminders as notifications

Reminders are exact Android alarms that fire with the app closed and after a reboot, with **Done** and **Snooze 1 Hour** buttons. See [Reminders](reminders.md#on-android).

## App Lock

**Settings, App Lock** requires your fingerprint, face, or device PIN before NoteTrace opens.

- **Lock NoteTrace** turns it on. You're asked to unlock once to confirm it works.
- **Lock After** sets how long the app can be in the background before it locks again: **Immediately**, **1 minute**, **5 minutes**, or **15 minutes**.

The lock also accepts your device PIN, pattern, or password, so a change to your enrolled fingerprints can't lock you out of your notes. If the phone has no fingerprint or face unlock set up, the device credential is used on its own.

App Lock is a screen lock for the app. Your notes are already encrypted at rest by Android's file-based encryption whenever the phone is locked; App Lock adds a prompt on an unlocked phone. It's a per-device setting and doesn't sync.

App Lock is separate from **biometric sign-in** on your profile page, which only unlocks a saved server session.

## Gestures

- **Swipe** a card sideways to archive it; **Undo** puts it back. In Archive, a swipe unarchives. Turn it off in Settings, Appearance, Swipe to Archive.
- **Long-press** a card to select it, then tap more cards to act on them together from the bar at the bottom. Keep holding and move your finger to drag the card to a new spot.
- **Pull down** at the top of the list to sync (connected to a server) or refresh the list.
- The bottom tab bar has Notes, Tasks, Reminders, Archive, Trash, and Settings. Search is the magnifier in the page header. On an unfolded foldable the tab bar gives way to a strip of icons beside the page.

## Foldables {#foldables}

On a foldable, the app adapts to the cover screen, the inner screen, and the fold itself: an icon strip instead of the tab bar when unfolded, each screen remembering its layout, the open note moving between full screen and the side pane as you fold and unfold, and layouts that keep content off the crease when the phone is half open. See [Foldables](features.md#foldables).

On a Samsung foldable, to keep NoteTrace open when you close the phone, turn on **Settings, Display, Continue apps on cover screen** for NoteTrace.

## Local mode

In local mode everything, including reminders, labels, images, version history, and imports, lives on the phone. Sharing needs a server, so it's hidden in local mode. Connecting to a server later uploads your local notes and photos.

## Related

- [How sync works](../mobile/sync.md)
- [Import and export](import-export.md)
