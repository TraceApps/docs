# Wear OS

CookTrace on a watch: the shopping list while your hands are full in an aisle, and the dish you're cooking while they're covered in flour. It talks to your server itself, so it keeps working with the phone in another room.

!!! note "Wear OS 3 and up"
    Built for Wear OS 3 and newer (Android 11 on the watch). Older watches run a different app model and aren't supported.

## Setting it up

1. **Install the watch app.** The APK is on the [Releases page](https://github.com/TraceApps/cooktrace/releases) alongside the phone one. Sideload it the usual way, or install from the phone with `adb -s <watch> install`.
2. **Open CookTrace on your phone** and sign in. That's the whole pairing: the phone sends the server address and a token to the watch over the standard Wear data connection, and the watch stores them.
3. **Open CookTrace on the watch.** Your shopping list appears.

There is nothing to type on the watch, and no separate account. Signing out on the phone takes the credentials off the watch too.

!!! warning "Both apps must be signed with the same key"
    If you build the apps yourself, sign the watch app with the same key as the phone app. Wear only carries data between apps that share a package name and a signing certificate, and when they differ, pairing silently does nothing.

## What's on the watch

**The shopping list.** Home, because it's the thing you use every week with a trolley in one hand. Grouped by aisle, in the order a shop is laid out. Tap an item to check it off.

**What you're cooking.** Press **Cook** on a recipe on your phone and it goes to the wrist, above the list. More than one can be on the go, which is the normal way to cook a meal: something in the oven while the next thing is started. Each one shows how many steps are done.

**Ingredients and steps.** Open a cook for its ingredients and its method, each as a checklist. Ticking is shared with the phone, so a step you marked on one is marked on the other.

**Timers.** A step that mentions a time offers a timer for it, and the watch runs up to eight at once, a pan and an oven. The watch buzzes at the end with a real alarm, so it happens with the app long gone from the screen, and says which dish it was rather than buzzing about nothing.

**The timer face.** One timer running gives you the whole screen: a ring that empties as the time goes, the time left, and `+1`, `+5` and Stop. Several gives you the list, soonest first, each in its own colour; tapping one opens it. Tapping never stops a timer, so a mis-tap can't throw away a braise.

**The colour.** Green while there's time, amber when it's getting on, red at the death, on the ring and on every row. A three hour braise isn't urgent at five minutes left, so the colour goes by the clock rather than by the fraction remaining.

**On the watch face.** While a timer runs it shows on the face and in the launcher's top row, counting down without the app open. With several running, the face follows the soonest and says "and 2 more"; tapping it opens the timers, where they're all listed. The system draws the counting, so nothing wakes up to tick it. Faces that don't support ongoing activities show it in the notification stream instead.

**I cooked this.** Finish a dish from the wrist and it goes in the cook diary, the same as pressing it on the phone.

**A tile.** One swipe from the watch face: what's left to buy, or what's counting down. Add it by long-pressing the watch face, then Tiles.

**A complication.** What's left to buy, on the watch face itself, tapping through to the app. Add it from your watch face's own customisation screen.

## Timers on both devices

While a cook is on the watch, the timers are shared: start, extend or stop one on either device and the other follows, and either can ring you, because the point is that you walked away from one of them.

This only happens during a cook. Pressing Cook on the phone is the explicit act that opens it; end the cook, or never hand one over, and nothing crosses in either direction. A watch in a drawer isn't woken for a timer you set on the phone and doesn't buzz about one.

## Without a connection

The watch keeps the last list and the recipes for the cooks you're in, so it opens and works in a shop or a basement kitchen with no signal. Anything you do while offline waits and goes up when the connection is back:

- checking items off the shopping list
- logging a dish as cooked

The screen says "Offline" or "2 waiting" while that's true, and shows the reason in red if the server refuses. Ticks collapse, so tapping twice sends once.

The tile and the complication read the same saved copy, so they show something sensible with no connection and no battery cost.

## Battery

Nothing polls. Timers are deadlines with alarms set for them rather than things counting, so a sleeping screen costs nothing, and every repeating thing on screen stops the moment your wrist drops. Ticking several things off the phone in a row wakes the watch once, not once per tap.

## What it doesn't do

Finding and editing recipes, the pantry, the cook diary and meal planning stay on the phone and the web app. Choosing what to cook belongs where you can read; the watch is what you touch once your hands are dirty.

## Related

- [Recipes](recipes.md)
- [Shopping list](shopping.md)
- [Cook diary](diary.md)
