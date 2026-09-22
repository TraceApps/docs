# Wear OS

LiftTrace on a watch: the session you are in, logging sets between them, the rest timer on your wrist, and finishing the workout without touching the phone. It talks to your server itself, so it keeps working with the phone in a locker.

!!! note "Wear OS 3 and up"
    Built for Wear OS 3 and newer (Android 11 on the watch). Older watches run a different app model and aren't supported.

## Setting it up

1. **Install the watch app.** The APK is on the [Releases page](https://github.com/TraceApps/lifttrace/releases) alongside the phone one. Sideload it the usual way, or install from the phone with `adb -s <watch> install`.
2. **Open LiftTrace on your phone** and sign in. That's the whole pairing: the phone sends the server address and a token to the watch over the standard Wear data connection, and the watch stores them.
3. **Open LiftTrace on the watch.** Today's session appears.

There is nothing to type on the watch, and no separate account. Signing out on the phone takes the credentials off the watch too.

!!! warning "Both apps must be signed with the same key"
    If you build the apps yourself, sign the watch app with the same key as the phone app. Wear only carries data between apps that share a package name and a signing certificate, and when they differ, pairing silently does nothing.

## What's on the watch

**The session.** Today's workout, in the order you do it, with a heading over every block: "Superset A" over a pairing you move between, "Standalone" over lifts that belong to no pairing. Each card says what it's due for next and how much is behind you, with an arrow on the one you're actually up to.

**Completed.** A block that's entirely done moves under a **Completed** heading at the bottom, keeping the order you did it in. A pairing only moves when the whole pairing is done, since half a superset is still work, and reopening a set brings it straight back up.

**Logging a set.** Tap the set you're on. The weight and reps are filled in from what the plan says or what you did last, and "Last time" shows what you lifted on this exercise before, which is the number you'd otherwise take your phone out to look up.

**The crown.** It scrolls, as it does everywhere else on a watch. Tap a number and it takes the crown: the number is outlined, and turning moves it. Tap it again to give the crown back to the list. Weight, reps, both sides of a unilateral lift and a hold can all be dialled this way.

**Rest.** It starts by itself after a set when your settings say so, and the row on the session screen counts it down in colour: green while there's time, amber when it's getting on, red at the death. The watch buzzes at the end with a real alarm, so it happens with the app long gone from the screen, and says "Rest over" on the wrist rather than buzzing about nothing. `+30` gives it longer, Skip ends it.

**On the watch face.** While a rest runs it shows on the face and in the launcher's top row, counting down without the app open. The system draws the counting, so nothing wakes up to tick it. Tapping it opens the timer. Faces that don't support ongoing activities show it in the notification stream instead.

**The session clock.** How long you've been training, started, paused or stopped from either device. Stopping writes the length onto the session.

**Finishing.** "Finish the session" ends it from the wrist: the length comes off the running clock, the day is marked done, and it goes up with everything else. With sets still unfinished it asks first and says how many, and they stay as they are.

**A hold.** A timed exercise, a plank, logs itself when the time is up, whether or not the screen is still on.

**A tile.** One swipe from the watch face: the set you're on, no app launch. Add it by long-pressing the watch face, then Tiles.

**A complication.** Sets done, on the watch face itself, tapping through to the app. Add it from your watch face's own customisation screen.

## The rest timer on both devices

A rest is one thing happening to one person, so both devices hold it and either can ring you. Start, extend or skip it on either and the other follows; whichever spoke last is the one that counts.

The phone only sends a rest to a watch whose app has been opened in the last half hour. A watch on a charger in another room isn't woken for one and doesn't buzz about it, and nothing is scheduled there. When the watch does have the rest, the phone keeps its own "Rest complete" notification to itself rather than mirroring it across, since the watch rings with its own alarm and its own words. With no watch in the picture, the phone's notification reaches your wrist the way it always did.

## Without a connection

The watch keeps the last session it loaded, so it opens and works in a gym with no signal. Anything you do while offline waits and goes up when the connection is back:

- logging, correcting, un-ticking and adding sets
- the session's length when the clock is stopped
- finishing the session

The screen says "Offline" or "2 waiting" while that's true, and shows the reason in red if the server refuses. A set changed twice is one entry, not two, because the last word wins; what goes up is the whole day with your changes replayed onto whatever the phone did in the meantime, so neither device overwrites the other.

The tile and the complication read the same saved copy, so they show something sensible with no connection and no battery cost.

## Battery

Nothing polls. The rest timer is a deadline with an alarm set for it rather than something counting, so a sleeping screen costs nothing, and every repeating thing on screen stops the moment your wrist drops. Over seven hours on a wrist with normal use, the watch app's share measures in seconds of CPU.

## What it doesn't do

Building and editing programs, exercise management, statistics, body stats, coaching and the radio all stay on the phone and the web app. The watch is for the part of a workout where your hands are busy.

## Related

- [Diary & logging](diary.md)
- [Programs](programs.md)
- [Settings reference](settings.md)
