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

**A checklist.** Tap an item, or its tick, to check it off. The whole row is a target, which matters on a small screen.

**A note.** Tap to read it. Long notes show their first stretch and say to open the phone for the rest; writing and editing belong where there's a keyboard.

**The shopping list.** Grouped by aisle in CookTrace's order, with amounts and the recipe an item came from. Check things off as you shop.

**Reminders.** What's coming up, soonest first, with "Today 6:30 PM", "Tomorrow", a weekday, or "Overdue" in red, and repeating ones marked. Tap the row to open the note, or the tick to be done with it. When a reminder actually fires, it arrives as a notification on the wrist with **Done** and **Snooze 1 Hour**.

**Speaking.** "Speak a note" on the main screen makes a new note, "Add an item" inside a checklist adds to it, and "Add to the list" on the shopping list sends it to CookTrace. The watch's own speech recognition does the listening, so nothing is recorded or uploaded and it follows the language your watch is set to.

**A tile.** One swipe from the watch face: what's due, or what's left to buy. Add it by long-pressing the watch face, then Tiles.

**A complication.** The same counts on the watch face itself, as "3 Due" or "7 Buy", tapping through to the app. Add it from your watch face's own customisation screen.

## Without a connection

The watch keeps the last lists it loaded, so it opens and works in a shop with no signal. Anything you do while offline waits and goes up when the connection is back:

- ticking items off and back on
- speaking a note, an item, or something to buy
- finishing a reminder

The screen says "Offline" or "2 waiting" while that's true. Ticks collapse, so tapping twice sends once, while spoken things never collapse: saying two items gives you two items.

The tile and the complication read the same saved copy, so they show something sensible with no connection and no battery cost.

## What it doesn't do

Writing and editing note text, images, files, drawings, voice recordings, sharing, labels, and search all stay on the phone and the web app. A watch is for checking something off and jotting a line, and the small screen is better for it when it isn't trying to be everything.

## Related

- [Reminders](reminders.md)
- [CookTrace shopping list](cooktrace.md)
- [Android app](android.md)
