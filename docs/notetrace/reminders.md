# Reminders

Any note can have one reminder: a date and time, and optionally a repeat.

## Setting a reminder

Tap the bell in the editor, on a card's hover actions, or in a card's long-press menu.

- **Later today**, **Tomorrow**, and **Next week** are one-tap presets.
- **Pick date and time** opens a date, a time, and a repeat: **Does not repeat**, **Daily**, **Weekly**, **Monthly**, or **Yearly**.

The reminder shows as a chip on the card and in the editor. Tap the chip to change it, or its **x** to remove it. The **Reminders** view lists notes with reminders, upcoming first (soonest at the top), then past one-off reminders.

## How repeats keep their time

A reminder is stored as its first occurrence in UTC, the repeat, and the time zone of the device that set it. Each next occurrence is worked out in that time zone, so an 8:00 AM daily reminder stays at 8:00 AM through daylight saving changes. A monthly reminder on the 31st falls on the last day of shorter months.

## Delivery

Reminders reach you three ways. Use any combination.

| Where | When it works | Turn it off |
|---|---|---|
| Android app | Always, even with NoteTrace closed or the phone restarted | Settings, Notifications, **Enable on This Device** (on the phone) |
| Browser | While NoteTrace is open in a tab, even in the background | Settings, Notifications, **Enable on This Device** (in that browser) |
| Push service (ntfy, Gotify, Apprise) and webhook | Always, sent by the server | Settings, Notifications, **Note Reminders** |

**Enable on This Device** is set per phone or browser, so you can have reminders on your phone but not on your work computer.

### On Android

The app schedules each reminder as an exact Android alarm, straight from the notes stored on the phone, so it fires on the minute with the app closed and after a reboot. When the alarm goes off, NoteTrace checks the note again first: a reminder you changed, cleared, or trashed on another device (and synced) won't fire.

The notification has two buttons:

- **Done** clears a one-off reminder. A repeating reminder keeps going.
- **Snooze 1 Hour** shows it again an hour later.

Tapping the notification opens the note. The app asks for notification permission the first time you set a reminder.

!!! note "Exact Reminder Times"
    Android lets reminder apps fire at the exact minute. If your phone has taken that permission away to save battery, Settings, Notifications shows **Exact Reminder Times** with an **Allow** button. Without it, reminders can arrive a few minutes late.

A reminder set on another device reaches the phone the next time NoteTrace syncs there. For reminders set elsewhere to reach you right away, add a push service as well.

### In the browser

With **Enable on This Device** on and browser permission granted (**Request Permission** under Settings, Notifications), NoteTrace shows a notification when a reminder comes due while it's open in a tab. Clicking it opens the note. Each reminder shows once per browser, even with several tabs open. Browsers can't show a notification for a page that isn't open, so use a push service for those.

### From the server

The server checks for due reminders every minute and sends each occurrence once through the push service set under **Settings, Notifications** ([ntfy](../integrations/ntfy.md), [Gotify](../integrations/gotify.md), or [Apprise](../integrations/apprise.md)). Turn **Note Reminders** off there to stop server pushes while keeping reminders on your devices.

- A reminder that came due while the server was down is still sent if the server is back within 2 hours. Older ones are skipped rather than arriving hours late.
- Each occurrence is recorded, so restarts never send one twice.
- Reminders in the trash aren't sent. Archived notes still remind you.
- Every delivered occurrence also fires the `reminder.fired` [webhook](webhooks.md), whether or not a push service is set.

!!! tip "More than one at once"
    With the Android app, an open browser tab, and a push service all on, the same reminder can arrive in each place. Turn off the ones you don't want: **Enable on This Device** per device, or **Note Reminders** for pushes.

## Shared notes

Reminders belong to the note's owner. People a note is shared with don't see or get its reminder. See [Sharing](sharing.md).
