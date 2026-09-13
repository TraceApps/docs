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

Reminders reach you two ways, and you can use either or both.

### On Android

The app schedules each reminder as an Android notification, so it fires on time even when NoteTrace is closed. Repeating reminders use Android's own repeat. The notification has two buttons:

- **Done** clears a one-off reminder. A repeating reminder keeps going.
- **Snooze 1 Hour** shows it again an hour later.

Tapping the notification opens the note. The app asks for notification permission the first time you set a reminder. The schedule is rebuilt whenever notes change or sync, so a reminder set on the web shows up on the phone after its next sync.

### From the server

The server checks for due reminders every minute and sends each occurrence once through the push service set under **Settings, Notifications** ([ntfy](../integrations/ntfy.md), [Gotify](../integrations/gotify.md), or [Apprise](../integrations/apprise.md)). Turn **Note Reminders** off there to stop server pushes while keeping reminders on the phone.

- A reminder that came due while the server was down is still sent if the server is back within 2 hours. Older ones are skipped rather than arriving hours late.
- Each occurrence is recorded, so restarts never send one twice.
- Reminders in the trash aren't sent. Archived notes still remind you.
- Every delivered occurrence also fires the `reminder.fired` [webhook](webhooks.md), whether or not a push service is set.

!!! tip "Both at once"
    If you use the Android app and a push service, you'll get a phone notification and a push for the same reminder. Turn off **Note Reminders** under Settings, Notifications if you only want the phone's.

## Shared notes

Reminders belong to the note's owner. People a note is shared with don't see or get its reminder. See [Sharing](sharing.md).
