# Coaching

LiftTrace has a first-class coach and trainee relationship built into the multi-user model. A trainer sees their assigned members' workouts, prescribes sessions on specific dates or as an open backlog, and leaves feedback per exercise or per workout. Trainees pull prescriptions into their Diary, complete them, and reply to notes. Nothing in the coaching flow is a paid tier, and everything lives on your own server.

## Roles

Three roles, set by an admin in Settings > User Management:

- **admin** everything: create users, assign roles, wire up OIDC, manage backups, and see every member and trainer.
- **trainer** the coach role. Sees only members assigned to them, prescribes their workouts, and leaves feedback.
- **member** the default. Sees their own diary, programs, and the Coaching tab if a trainer has been assigned.

One member can have exactly one primary trainer (the `trainer_id` foreign key). A trainer can have many members. If a member needs to switch coaches, an admin unassigns and reassigns from User Management. The first-created account on a fresh install is always `admin`.

## Assigning a trainer to a member

Consent is admin-managed. LiftTrace does not have a self-service "invite a client" or "request my coach" flow between users; the person with the admin badge decides who works with whom. Two paths:

1. **Direct assign.** Settings > User Management > pick a user > **Assign trainer** > choose from the trainer list. Instant. The member sees the Coaching tab appear on their next navigation.
2. **Trainer-claim.** A trainer opens the Coaching tab and sees an **Unassigned members** list of every member with no trainer yet (`GET /api/trainer/unassigned-members`). Tap **Claim** and the trainer becomes that user's `trainer_id`. Refused if someone else has already claimed them; admins can reassign to break ties.

To invite a person who does not have an account yet, use the general **Invite User** flow (Settings > User Management > **Invite User**), which sends an email invite via SMTP or produces a shareable link. Once they accept and land as a `member`, use one of the two paths above to assign a trainer.

## The trainer view

Coaching tab, trainer side: a roster of assigned members with each member's latest workout, active program, and 7-day workout count. Tap a member to see their full workout history, body stats, PRs, and the Prescriptions and Feedback panels.

- **Recent workouts** for the selected member. Each card carries an indicator when the trainer has already left feedback on it, and a second indicator when the member has replied.
- **Feedback threads.** Open a completed workout and drop a note either at the workout level or on a specific exercise position within it. The position is by index (not by exercise ID), so if the same exercise appears twice in one workout each surface gets its own thread. Empty note deletes the thread. Notes are single-reply (bidirectional): the member sees the note, taps to acknowledge, and can post one reply.
- **Prescriptions.** Send the member a workout to do. Two flavors:
    - **Dated** the prescription shows in the member's Diary on that exact date.
    - **Undated** the prescription sits in the member's Coaching inbox and can be pulled into any day.

Prescriptions can be built from an existing template (`template_id`) or by hand-writing name plus exercises. See `POST /api/trainer/members/:id/prescriptions` for the shape.

## The member view

Coaching tab, member side: prescriptions inbox (upcoming dated ones plus undated backlog), feedback inbox (unread first), and reply thread on each note. Dated prescriptions also appear inline in the Diary on their day, with the trainer's name on the card. See [Diary and set logging](diary.md#coach-prescriptions).

## Activity feed

The trainer dashboard carries a chronological **Activity** feed of what has happened across all assigned members. Three kinds:

- `prescription_completed` a member logged the workout you prescribed on the date it was due.
- `prescription_missed` the date passed and the prescribed template was not logged. Fired by the 15-minute scheduler.
- `feedback_reply` a member replied to one of your notes.

Rows are marked seen with `POST /api/trainer/activity/seen`; unread counts drive the Coaching tab badge.

## Dropping a member

A trainer can drop a member from the Coaching tab (`DELETE /api/trainer/members/:id`). That clears the trainer link, deletes prescriptions the trainer had sent to that member, and removes any program assignments the trainer created for them. The member keeps every workout they logged. To reassign, an admin picks it up from User Management.

There is no "trainee revokes coach" endpoint. If a member wants a different coach or none, the admin handles it: unassign the trainer or reassign to a different one.

## Notifications for coaches

Trainer-side notification toggles live in Settings > App > Notifications:

- **Member completes prescribed** push when someone logs a workout you prescribed on the correct day.
- **Member missed a prescription** push when the scheduler marks a dated prescription as missed.
- **Member replied to feedback** push when a note gets a reply.

Push delivery uses the same Gotify / ntfy / Apprise pipeline every other alert uses. See the Notifications section of [Settings reference](settings.md#notifications).

## Assigning a program instead of a workout

If you want a member to follow a whole training block rather than one session at a time, build the program, then assign it: `POST /api/programs/:id/assign` (see [Programs and templates](programs.md#trainer-authored-programs)). The member sees the program in their Programs list and can activate it. Edits you make propagate on their next load.

## Related

- [Programs and templates](programs.md)
- [Multi-week progression](progression.md)
- [Diary and set logging](diary.md)
- [Settings reference](settings.md)
