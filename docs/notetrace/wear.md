# Wear OS

NoteTrace on a watch: your notes and checklists, the CookTrace shopping list, and what's due, without reaching for the phone. It talks to your server itself, so it keeps working when the phone is in another room or in a bag.

!!! note "Wear OS 3 and up"
    Built for Wear OS 3 and newer (Android 11 on the watch). Older watches run a different app model and aren't supported.

## Setting it up

1. **Install the watch app.** The APK is on the [Releases page](https://github.com/TraceApps/notetrace/releases) alongside the phone one. Sideload it the usual way, or install from the phone with `adb -s <watch> install`.
2. **Open NoteTrace on your phone** and sign in. That's the whole pairing: the phone sends the server address and a token to the watch over the standard Wear data connection, and the watch stores them.
3. **Open NoteTrace on the watch.** Your notes appear.

There is nothing to type on the watch, and no separate account. Signing out on the phone takes the credentials off the watch too.

!!! warning "Both apps must be signed with the same key"
    If you build the apps yourself, sign the watch app with the same key as the phone app. Wear only carries data between apps that share a package name and a signing certificate, and when they differ, pairing silently does nothing.

## What's on the watch

**The list.** Pinned notes first, then the rest, newest first, each showing either how many items are left or the first lines of its text. The CookTrace shopping list sits near the top when CookTrace is linked, and a Reminders row appears above it when anything is scheduled.

**A checklist.** Tap an item, or its tick, to check it off; the whole row is a target, and it buzzes so you can tell it took. Ticked things drop to the bottom under a heading that counts them, dimmed and struck through, so you can see what you just did and put it back if it was the wrong one. The crown scrolls every screen.

**A note.** Tap to read it. Long notes show their first stretch and say to open the phone for the rest; writing and editing belong where there's a keyboard.

**The shopping list.** Grouped by aisle in CookTrace's order, with amounts and the recipe an item came from. Check things off as you shop.

**Reminders.** What's coming up, soonest first, with "Today 6:30 PM", "Tomorrow", a weekday, or "Overdue" in red, and repeating ones marked. Tap the row to open the note, or the tick to be done with it. When a reminder actually fires, it arrives as a notification on the wrist with **Done** and **Snooze 1 Hour**.

**Speaking.** "Speak a note" on the main screen makes a new note, "Add an item" inside a checklist adds to it, and "Add to the list" on the shopping list sends it to CookTrace. The watch's own speech recognition does the listening, so nothing is recorded or uploaded and it follows the language your watch is set to.

**A time you say becomes the reminder.** "Call the plumber tomorrow at nine" makes a note that says "call the plumber", due at nine tomorrow, rather than writing the time into the text. It understands the phrases people actually say to a watch: in twenty minutes, tonight, tomorrow at nine, Friday at eight, at 6:30 pm. A time already past today is taken as tomorrow. This is worked out on the watch, so it still happens with no connection, and the note goes up later with the time it was meant to have. Anything more involved is Trace's job, on the phone.

**Refresh.** The app asks the server every time you open it. The Refresh row at the bottom of the list asks again without leaving and coming back.

**A tile.** One swipe from the watch face: what's due, or what's left to buy. Add it by long-pressing the watch face, then Tiles.

**A complication.** The same counts on the watch face itself, as "3 Due" or "7 Buy", tapping through to the app. Add it from your watch face's own customisation screen.

## Choosing what's on the watch

With a big library, the watch list gets long. On the phone, **Settings, Notes, On Your Watch** narrows it:

- **Everything** is the default, and nothing changes until you pick something.
- **Only what I pick** gives each note and checklist a switch. The heading says where you stand: "4 notes on the watch", or "Nothing picked yet, so the watch shows everything".
- Clearing every pick puts it back to everything, so the watch can't end up empty by accident.

The narrowing happens on the server, so a shorter list is also a smaller download. Reminders and the shopping list are never hidden by this: a reminder is time-sensitive, and there's only one shopping list.

## Without a connection

The watch keeps the last lists it loaded, so it opens and works in a shop with no signal. Anything you do while offline waits and goes up when the connection is back:

- ticking items off and back on
- speaking a note, an item, or something to buy
- finishing a reminder

Each of those says so as you do it ("Note saved", "Reminder set", "Added"), and adds "waiting for a connection" when there's nothing to send it over. The screen says "Offline" or "2 waiting" while that's true, and shows the reason in red if the server refuses. Ticks collapse, so tapping twice sends once, while spoken things never collapse: saying two items gives you two items.

The tile and the complication read the same saved copy, so they show something sensible with no connection and no battery cost.

## What it doesn't do

Writing and editing note text, images, files, drawings, voice recordings, sharing, labels, and search all stay on the phone and the web app. A watch is for checking something off and jotting a line, and the small screen is better for it when it isn't trying to be everything.

## Related

- [Reminders](reminders.md)
- [CookTrace shopping list](cooktrace.md)
- [Android app](android.md)
