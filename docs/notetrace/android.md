# Android: share sheet and App Lock

The NoteTrace Android app is the same app as the web version, plus a few things only a phone can do. Install and mode selection work like the other Trace apps: see [Install the Android app](../mobile/install.md) and [Local vs server-connected mode](../mobile/modes.md).

## Save from any app

Share text or a link from any Android app (a browser, a chat, a news reader) and pick **NoteTrace** in the share sheet. NoteTrace opens a new note with the shared text in it, and a shared page title becomes the note's title. It saves on its own; close it or keep typing.

The installed web app (PWA) accepts shares the same way on platforms that support the Web Share Target API, such as Chrome on Android.

!!! note
    Sharing images into a note isn't supported yet.

## Reminders as notifications

Reminders are scheduled as Android notifications on the device, with **Done** and **Snooze 1 Hour** buttons. See [Reminders](reminders.md#on-android).

## App Lock

**Settings, App Lock** requires your fingerprint, face, or device PIN before NoteTrace opens.

- **Lock NoteTrace** turns it on. You're asked to unlock once to confirm it works.
- **Lock After** sets how long the app can be in the background before it locks again: **Immediately**, **1 minute**, **5 minutes**, or **15 minutes**.

The lock also accepts your device PIN, pattern, or password, so a change to your enrolled fingerprints can't lock you out of your notes. If the phone has no fingerprint or face unlock set up, the device credential is used on its own.

App Lock is a screen lock for the app. Your notes are already encrypted at rest by Android's file-based encryption whenever the phone is locked; App Lock adds a prompt on an unlocked phone. It's a per-device setting and doesn't sync.

App Lock is separate from **biometric sign-in** on your profile page, which only unlocks a saved server session.

## Local mode

In local mode everything, including reminders, labels, version history, and imports, lives in the on-device SQLite database. Sharing needs a server, so it's hidden in local mode. Connecting to a server later uploads your local notes.

## Related

- [How sync works](../mobile/sync.md)
- [Import and export](import-export.md)
