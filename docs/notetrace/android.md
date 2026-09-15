# Android: share sheet, voice notes, and App Lock

The NoteTrace Android app is the same app as the web version, plus a few things only a phone can do. Install and mode selection work like the other Trace apps: see [Install the Android app](../mobile/install.md) and [Local vs server-connected mode](../mobile/modes.md).

## Save from any app

Share text, a link, or photos from any Android app (a browser, a chat, the gallery, a news reader) and pick **NoteTrace** in the share sheet. NoteTrace opens a new note with the shared content in it: text goes in the body, a shared page title becomes the note's title, photos are added as images (up to 20 at a time), and audio (a recording from a voice recorder app, say) becomes a [voice note](voice-notes.md). It saves on its own; close it or keep typing.

The installed web app (PWA) accepts shared text, links, photos, and audio the same way on platforms that support the Web Share Target API, such as Chrome on Android.

## Home screen shortcuts

Long-press the NoteTrace icon for **Voice Note**, **New Note**, and **New List**. Each opens the app straight into a new note; Voice Note starts recording. The installed web app offers the same shortcuts on Android.

## Voice notes

The Android app records voice notes itself rather than through the web view, so recording keeps going with the screen off or another app open, for up to 3 hours. While it records, a notification shows the time with **Pause** or **Resume** and **Stop**; stop it there and the recording is saved to the note when you come back. A recording too long for your transcription service is split on the phone in local mode. See [Voice notes](voice-notes.md).

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
- The bottom tab bar has Notes, Reminders, Tasks, Archive, Trash, and Settings. Search is the magnifier in the page header. On an unfolded foldable the tab bar gives way to a strip of icons beside the page.

## Foldables {#foldables}

On a foldable, the app adapts to the cover screen, the inner screen, and the fold itself: an icon strip instead of the tab bar when unfolded, each screen remembering its layout, the open note moving between full screen and the side pane as you fold and unfold, and layouts that keep content off the crease when the phone is half open. See [Foldables](features.md#foldables).

On a Samsung foldable, to keep NoteTrace open when you close the phone, turn on **Settings, Display, Continue apps on cover screen** for NoteTrace.

## Local mode

In local mode everything, including reminders, labels, images, version history, and imports, lives on the phone. Sharing needs a server, so it's hidden in local mode. Connecting to a server later uploads your local notes and photos.

## Related

- [How sync works](../mobile/sync.md)
- [Import and export](import-export.md)
