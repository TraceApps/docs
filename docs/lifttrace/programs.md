# Programs and templates

A **program** in LiftTrace is a named training block: a goal, a list of workout templates, an optional duration in weeks, and, if activated, the source-of-truth prescription for what your Diary prefills every session. Templates are the individual workouts (Push A, Pull B, Legs) that programs group together.

## Goal-based programs

Every program carries a **goal** tag: `strength`, `hypertrophy`, `cutting`, `bulking`, `endurance`, or `general`. The tag is used for filtering in the Programs list and helps Trace answer questions like "what should I focus on this block". The goal does not gate features; you can absolutely run a hypertrophy program during a cut.

## Starter templates

On first boot the app seeds a set of templates and programs so the Programs page is not empty:

- **Push / Pull / Legs**
- **Upper / Lower**
- **Full Body 3x**
- **Monumental Valley** (branded program)
- **Shredville** (branded program)

Seeds live in `server/seed-templates.js` and `server/seed-programs.json`. Delete or edit any of them freely; they are seeds, not lock-ins.

## The active program

Exactly one program is active per user. On the Programs page, tap **Activate** on the one you want to follow; the previous active program deactivates automatically. The Diary then does two things:

1. On an empty day, shows a **Load today's workout** button that resolves to the correct template.
2. When you load that template, prefills every exercise, target weight, target reps, warm-up marker, and RPE target from the prescription.

Deactivate from the same button, or from `POST /api/programs/deactivate` if you are wiring it externally.

## Multi-week progression plans

Set a program's **Duration (weeks)** in the editor and the Workout template gains a **Week tab strip** across the top. Each week gets its own per-exercise matrix:

- **Sets**: how many
- **Reps**: target rep count or range (`6-8`)
- **Tempo**: eccentric / pause / concentric / pause (e.g. `3010`)
- **Rest (s)**: per-exercise rest to feed the timer
- **Load**: absolute weight, percentage of a reference, or an RPE target

A "copy this week to next" shortcut clones the current column so you only edit the changes.

The Diary resolves which week you are on using the program's **advance mode**:

- `sessions` (default): the week cursor moves forward after you complete the last workout in a week.
- `calendar`: the cursor moves by real days once a week has elapsed.

On the last week, choose `hold` (stay on the final week's prescription) or `repeat` (loop back to week one). You can nudge the cursor manually via **Repeat this week** or **Advance to next week** in the Load Workout sheet, or programmatically via `POST /api/programs/:id/week-cursor`.

The week is stamped onto the workout at load time, so a workout you logged in week 2 keeps its week label even after you advance to week 3. Charts and history reflect the actual week you trained under.

## How templates prefill the Diary

Templates carry per-set specs, not just per-exercise targets. Every set row in a template can mark warm-up, unilateral L/R, an RPE target, and a rep count. Load the template and every one of those settings flows into the diary. Inside a programmed week, the week's matrix overrides last-session auto-fill so the plan wins.

Templates you edit inline in the Program editor use the same set-row component as the Diary itself (`SetRow.svelte`), so a set spec you build in the template looks and behaves like the set you will log.

## Creating a custom program

Programs page > **+ New Program**. Fill in name, goal, and (optionally) duration in weeks. Add workout templates from the picker (search across all templates you and your trainer have built), or create new templates inline.

For each template:

1. Pick exercises from the library or search by name.
2. Set the per-exercise Sets / Reps / Tempo / Rest / Load.
3. Mark warm-up sets, RPE targets, or unilateral splits per set row.
4. Group into supersets by assigning the same superset ID to paired rows.

Reorder templates within a program with the drag handle or arrow buttons (`PUT /api/programs/:id/reorder`). Duplicate a template from its menu when you want a variant. Assign a program to another user (trainer-only) via `POST /api/programs/:id/assign` (see [Coach and trainee model](coaching.md)).

## Trainer-authored programs

Trainers can build a program and assign it to specific members. The assigned member sees it in their Programs list alongside their own, and can activate it like any other. If the trainer edits the program, the change propagates on the member's next load.

Prescribing a single workout on a single day is different from assigning a whole program; see [Diary and set logging](diary.md#coach-prescriptions) for the day-of-the-week flow.

## Import / export

Program export and import are not on the current release. If you want to publish a program for the community today, the workaround is per-exercise sharing (Exercise detail > Share button) plus a written description of the template. Full program bundles are on the roadmap.

## Related

- [Diary and set logging](diary.md)
- [Exercise library](exercises.md)
- [Coach and trainee model](coaching.md)
