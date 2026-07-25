# Diary and set logging

The Diary is where every workout starts and ends. This page walks through the set-logging model, the parser and prefill helpers that save typing, and the ergonomics (rest timer, supersets, warm-ups) that separate a real lifting log from a spreadsheet.

## Anatomy of a set

Every set row carries:

- **Weight** and **reps** as separate inputs. Weight units follow your profile setting (lbs or kg).
- A **completion checkbox**. Checking it stamps the set as done, starts the rest timer if enabled, and gates superset-round completion.
- **RPE** (Rate of Perceived Exertion). Enable per-set RPE tracking in Settings > Workout. Values 1 to 10, half-steps allowed.
- **Warm-up toggle** on the row menu. Warm-ups are excluded from volume, PR detection, rest-timer firing, auto-collapse, and superset-round gating.
- **Unilateral L/R split** on exercises that ask for it (single-arm rows, split squats). Reps become two inputs.
- **Set-number override** in supersets, so a paired A1/A2/B1 flow keeps the round math right.

Sets you add on the fly through the row menu inherit the previous set's weight and reps. Empty sets never save; empty workouts auto-delete server-side.

Writes are debounced by 350 ms with optimistic UI. Pending writes flush on `pause`, `visibilitychange`, and `pagehide`, so a set entered right before backgrounding the phone hits the server or the offline queue instead of dying with the timer (fix in v1.0.1).

## Warm-ups

Warm-up sets are first-class, not free-text notes. Toggle **Auto-Generate Warm-Up Sets** in Settings > Workout and the app inserts a warm-up ramp for every working set you enter, based on the working weight and unit. Generator lives in `src/lib/workout.js` (`generateWarmupSets`).

You can also mark any set as warm-up from its row menu. Once marked, it stops counting toward volume, PRs, the completion summary, and the rest timer. Superset-round gating skips it too.

## Smart Add and the natural-language parser

The **Smart Add** button (also the sparkle FAB and the 700 ms hold on the Trace FAB) opens a text field that turns prose into structured sets. Type or paste something like:

```text
bench 3x5 @ 225
A1: curls 3x12 @ 30
A2: pushdowns 3x12 @ 40
squat 5x5 @ 315 RPE 8
pullups 3xAMRAP @ BW
overhead press 4x8 range 6-8 @ 95
```

The parser handles:

- **Cross-multiplied set specs**: `3x5 @ 225` and `5x5 @ 315`.
- **RPE targets**: `RPE 8`, `@8`.
- **Rep ranges**: `range 6-8`, `6-8 reps`.
- **AMRAP**: `AMRAP`, `x AMRAP`.
- **Bodyweight**: `@ BW`, `bodyweight`.
- **Supersets**: prefixes `A1:`, `A2:`, `B1:`, `B2:` group into paired exercises. Same letter = same superset.
- **Aliases**: `BB`, `DB`, `BP`, `OHP`, `DL`, `SQ`, `RDL` map to their full names.
- **Fuzzy name matching** against your exercise library: exact, then starts-with, then substring, then token overlap.

A preview screen shows what got parsed, what matched, and what did not. Fix names in place, then commit.

Voice hold-to-record uses the same parser downstream. Press and hold the Trace FAB for 700 ms, speak the workout, release. The Web Speech API or the native recognizer transcribes; the transcript feeds Smart Add and opens the review sheet.

## Progressive-overload prefill

Auto-fill Last Weights (default on; Settings > Workout) prefills weight and reps on new sets from your last completed working set for that exercise. Warm-ups are ignored for the calculation.

If the diary loaded from a program template, the template's prescription wins over last-session auto-fill. Inside a multi-week programmed block, the current week's target Sets, Reps, Tempo, Rest, and Load matrix takes precedence.

## Program template prefill

Load a workout from your active program (**Load today's workout** button on an empty day, or from the Programs page) and the diary spawns with every exercise, target rep count, target weight, warm-up marker, and RPE target from the template already in place. Add or remove sets freely; the prescription is a starting point.

## Coach prescriptions

If a trainer has prescribed today's workout, it lands in Diary on the prescribed day with the trainer's name on the card. Dated prescriptions show only on the assigned day; undated ones sit in the Coaching inbox until you pull one into the diary. See [Coach and trainee model](coaching.md).

## Last-workout quick-load

Empty days show a horizontal row of quick-load cards for recent workouts. One tap prefills the whole session. Handy for "same as last Push day" without opening the programs screen.

## Rest timer

Countdown from a configurable per-set duration (Settings > Workout > Rest Duration), with per-set overrides on the set row. Auto-start on set completion is a separate toggle. Vibrate and tone are separate toggles so you can silence one without the other.

Beeps schedule via `setTimeout`, not a tick loop, so background throttling does not drop the cue. On Android the timer survives backgrounding through a native alarm (`RestTimerCuePlugin.java`) that posts a notification-channel cue at the target time.

Superset gating: in a superset the timer waits until every exercise in the round has the same completed-set count before firing. Warm-ups are excluded from the round math.

## Supersets

Visual cards group paired exercises. Sets inside the card are round-aware: A1 set 2 and A2 set 2 belong to the same round. Round-number pickers on each row let you correct the app when a set sits out or gets reshuffled.

Rest timer fires between rounds only, not between the exercises inside a round. Auto-collapse folds a completed superset the same way it folds an individual exercise.

## AMRAP, RPE, ranges, bodyweight

- **AMRAP**: set the reps field to `AMRAP` (or `A`). PR detection treats the top rep count as the score.
- **RPE**: enable per-set RPE in Settings > Workout. Trace and the completion summary annotate sets with the RPE value.
- **Rep ranges**: templates carry ranges (`6-8`). The diary shows the range as the target; the actual field takes the number you hit.
- **Bodyweight**: `@ BW` in Smart Add, or leave weight blank on any exercise marked as bodyweight in the library.

## Music during a session

When the Radio player is active, its mini-bar stays visible across every screen so you can adjust playback without leaving the diary. Full detail in [Radio player](radio.md).

## Related

- [Programs and templates](programs.md)
- [Exercise library](exercises.md)
- [Radio player](radio.md)
- [Trace in LiftTrace](trace.md)
