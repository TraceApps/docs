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
- `calendar`: the cursor moves by real days: seven days after the program's start date puts you on week 2 regardless of how many sessions you actually did.

`sessions` is what most people want because a missed week does not silently push you into a harder prescription. `calendar` matches strict block periodization where the date matters more than the count.

## What happens after the last week

Set **On complete** on the program:

- `hold` (default): stay on the final week's prescription indefinitely. Good for maintenance blocks or an open-ended top set.
- `repeat`: loop back to week 1. Good for a rolling cycle you want to keep running.

## Nudging the cursor

Life happens: an injury, a travel week, a wave you want to run twice. The Load Workout sheet in the Diary has two buttons for that:

- **Repeat this week** pins the cursor to the current week so the next few sessions keep pulling week N's prescription.
- **Advance to next week** jumps forward one week without waiting on session count.

Both write to the assignment via `POST /api/programs/:id/week-cursor` and both auto-advance from the pin using the same clock the program uses, so repeating week 2 does not strand you there forever.

## Week stamping and history

The week you were on when a workout loads is stamped onto that workout record at load time. It is not derived from the current cursor after the fact. A session you logged in week 2 keeps its `week 2` label after you advance to week 3, so charts, PR history, and coach reports reflect the actual week you trained under.

The resolution logic lives in [`server/lib/programWeek.js`](https://github.com/TraceApps/lifttrace/blob/main/server/lib/programWeek.js) if you want to read the code.

## Completing sets to advance

Progression counts on completed working sets. Mark sets done as you finish them (the checkbox on each row) and the session tallies toward the next week. Warm-ups are ignored for advance-mode counting the same way they are ignored for PRs and volume: see [Diary and set logging](diary.md#warm-ups). Empty sets do not count and empty workouts auto-delete, so a session you opened and never used will not trip the advance.

## Related

- [Programs and templates](programs.md)
- [Diary and set logging](diary.md)
- [Coaching](coaching.md)
