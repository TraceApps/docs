# Multi-week progression

A single template is fine for daily lifting, but real strength programs run in blocks: week 1 primes the pattern, week 2 adds a bit, week 3 pushes, week 4 deloads. LiftTrace bakes that shape into any program via a per-week Sets / Reps / Tempo / Rest / Load matrix inside every workout template. This page covers how to build one, how the current week resolves, and how to steer the cursor when the plan and reality drift apart.

For the basics of programs and templates (goal tags, activating one, coach-assigned programs), start at [Programs and templates](programs.md). This page picks up from there.

## Turning on multi-week mode

Open a program in the editor and set **Duration (weeks)** to something greater than 1. Every workout template inside the program now grows a **Week tab strip** across the top of the exercise list. Each tab holds its own matrix:

- **Sets** target set count for that exercise this week.
- **Reps** exact number or a range (`6-8`).
- **Tempo** four-digit eccentric / pause / concentric / pause (`3010`).
- **Rest (s)** per-exercise rest that also feeds the timer.
- **Load** absolute weight, a percentage of a reference, or an RPE target.

The matrix is per-exercise per-week, so week 1 squat can be `3x5 @ 70%` and week 3 squat can be `3x3 @ 82.5%` in the same template without duplicating the workout.

## Building the matrix

There is no linear-add or percentage-add automation; every week is authored by hand. The gain from that is total flexibility: a peaking block, a wave-load cycle, or a deload week that swaps in different accessories all live in the same template. The cost is typing.

To keep the typing down, each week tab has a **Copy this week to next** shortcut that clones the current column into the next one. The usual workflow: fill in week 1, copy to week 2, bump the numbers, copy to week 3, bump again, then hand-write week 4 as a deload. Three edits instead of four full setups.

## How the current week resolves

The Diary asks the program which week to prefill from every time you load today's workout. The answer depends on **Advance mode**, set on the program itself:

- `sessions` (default): the cursor moves forward once you have completed the last workout in the current week. If your program has three workouts a week and you have logged three since the last advance, next Diary load steps to week 2.
- `calendar`: the cursor moves by real days: seven days after the program's start date puts you on week 2 regardless of how many sessions you actually did. The week moves at midnight where you are. [Excused days](diary.md#excused-days) pause it (below).

Without user accounts, a program starts the day you choose it as the one you follow (from the next release; before, a `calendar` program there stayed on week 1, or on the week you chose). Choosing it again doesn't restart it; stopping it and choosing it later does. One you were already following when you updated counts from its first finished workout since you last trained another program, or from the update if it has none. Turning user accounts on or off keeps the program you follow, its start and the week you chose, and the Android app connected to the server shows the same week.

`sessions` is what most people want because a missed week does not silently push you into a harder prescription. `calendar` matches strict block periodization where the date matters more than the count.

## Excused days pause a calendar program

A day you [excuse](diary.md#excused-days) because you couldn't train (injured, ill, an appointment) pushes a `calendar` program back one day, so a week off sick doesn't skip you a week ahead in the plan. It happens by itself, for your own programs and for one your coach assigned:

- **Which days count.** Each excused day from the program's start date (or the day it was assigned, or started without accounts, when it has none) through today, where you are. A day you excuse ahead of time counts once it comes.
- **Giving the day back.** Remove the excuse, or train that day anyway (a completed working set, or cardio with Track Cardio on), and the day counts again.
- **Where it shows.** The program's card, the program page and the week bar in the Load Workout sheet say **Paused 5 days for excused days** under the week. Your coach sees the same week and the same line on your page.
- **Choosing a week yourself** (below) starts the clock over from then: only days excused from that day on pause it.
- **On the last week**, with `hold`, a day excused once you're on it doesn't count: the week can't move any more, so the count doesn't grow. A day that kept you out of the last week still counts. With `repeat` every one does.
- **`sessions` programs** are never paused: they only move when you train.

`get_active_program` (MCP) and `GET /api/v1/programs/active` carry the count as `paused_days`, already taken off `current_week`, counted to your today in the time zone your app last reported to the server (from the next release; before, the server's).

## What happens after the last week

Set **On complete** on the program:

- `hold` (default): stay on the final week's prescription indefinitely. Good for maintenance blocks or an open-ended top set.
- `repeat`: loop back to week 1. Good for a rolling cycle you want to keep running.

## Nudging the cursor

Life happens: an injury, a travel week, a wave you want to run twice. The Load Workout sheet in the Diary has two buttons for that:

- **Repeat this week** pins the cursor to the current week so the next few sessions keep pulling week N's prescription.
- **Advance to next week** jumps forward one week without waiting on session count.

Both write to the assignment via `POST /api/programs/:id/week-cursor` and both auto-advance from the pin using the same clock the program uses, so repeating week 2 does not strand you there forever. For a `calendar` program, days excused after the pin still pause it; earlier ones no longer count. **Auto** goes back to the program's own clock, with every excused day since its start.

## Week stamping and history

The week you were on when a workout loads is stamped onto that workout record at load time. It is not derived from the current cursor after the fact. A session you logged in week 2 keeps its `week 2` label after you advance to week 3, so charts, PR history, and coach reports reflect the actual week you trained under.

The resolution logic lives in [`server/lib/programWeek.js`](https://github.com/TraceApps/lifttrace/blob/main/server/lib/programWeek.js) if you want to read the code.

## Completing sets to advance

Progression counts on completed working sets. Mark sets done as you finish them (the checkbox on each row) and the session tallies toward the next week. Warm-ups are ignored for advance-mode counting the same way they are ignored for PRs and volume: see [Diary and set logging](diary.md#warm-ups). Empty sets do not count and empty workouts auto-delete, so a session you opened and never used will not trip the advance.

## Related

- [Programs and templates](programs.md)
- [Diary and set logging](diary.md)
- [Coaching](coaching.md)
