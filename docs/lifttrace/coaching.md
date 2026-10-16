# Coaching

A coach and a client on one LiftTrace server: the coach asks, the client accepts, and from then on the coach sees the client's workouts, prescribes sessions, leaves notes on workouts and exercises, and talks with them in one conversation. Either side can end it at any time. Nothing in coaching is a paid tier, and everything lives on your own server.

Coaching needs accounts on a server. The Android app without a server says **Coaching Needs a Server** on the Coach and Coaching tabs, with a button to connect one.

## Roles

Three roles, set by an admin in Settings, Users:

- **Admin**: manages accounts, roles and pairings. An admin can also coach, but sees a client's data only as that client's coach.
- **Trainer**: the coach role. Can ask people to be their clients and coach the ones who accept.
- **Member**: the default for everyone else. A member can be coached.

A client has one coach at a time; a coach can have many clients. The first account on a fresh install is always an admin.

## Asking someone to be your client

On **Programs**, **Coaching**, tap **Add**. Pick a person (the list shows names only) and add a note if you like, then **Send Request**. Nothing of theirs is shared until they accept. Your request shows under **Requests** as **Waiting for an Answer**; you can **Cancel Request** while it waits. A declined request, or a client who leaves, shows there for a week.

A request also comes from:

- an admin choosing a trainer for a member in **Settings, Users** (the picker sends that trainer's request);
- choosing a **Coach** when creating or inviting a member in **Settings, Users**: the request comes from that coach (an admin can choose themselves) and waits for the new account's first sign-in.

An account created or invited with no coach chosen gets no request.

Asking never tells you whether someone already has a coach.

## Answering a request

The person asked sees a dot on **Programs** and a **Coach** tab there, with a card: who wants to be their coach, what that coach will see (your workouts, the programs you follow and your set videos) and what they can do (assign programs, prescribe workouts, leave notes). **Accept** or **Decline**. To accept a second coach, leave the first one; the card says so.

## Leaving, and ending coaching

- The client leaves from their coach's name on the **Coach** tab: **Leave Coach**.
- The coach ends it from the client's page in Coaching (**End Coaching**).
- An admin ends any pairing by setting the member's trainer to **No trainer** in Settings, Users.

When the client leaves or the coach ends it, the prescriptions and programs that coach gave the client are removed, as before. When an admin ends it, or a role change does, they stay. Either way the coach loses access at once. The conversation stays readable for the client, under **Past Coaching**.

## The coach's side

**Programs**, **Coaching** lists your clients with their latest workout, active program, workouts this week and unread messages. Open a client for:

- **Conversation**: see below.
- **Programs**: assign one of your programs, make it their active one, or remove it. The client can follow it but not change it.
- **Prescribed Workouts**: what you prescribed, with the client's answer (**Waiting for an Answer**, **Accepted**, **Declined**) and whether it was completed or missed.
- **Recent Workouts**: open a completed workout to read it and leave a note on the workout or on any exercise. Each note gets one reply from the client. Filmed sets carry a **Watch** chip ([Set videos](set-videos.md)).

**Recent Activity** on the Coaching list shows your own clients' events: a prescribed workout completed or missed, and replies to your notes.

## The conversation

One conversation per coach and client, beside the notes on workouts and exercises (which stay where they are). The client finds it on the **Coach** tab, the coach on the client's page in Coaching.

- Text up to 4,000 characters and one photo per message. Photos are served only to the two of you (the coach only while coaching), and where a photo was taken is not sent.
- Read receipts, an unread count on the Coach and Coaching tabs, and a dot on Programs.
- Delete your own message; the other person sees "Message deleted".
- It is kept on the device and opens with no connection. A message written offline shows as waiting and is sent once when the connection returns. While a conversation is on screen it checks for news every 10 seconds; nowhere else does.
- Trace never reads it.

## Prescriptions

A coach sends a workout to do, from one of their templates or written out, on a date or for any day. The client answers first:

- On the Diary, a prescribed workout shows **Accept** and **Decline**: on its day in the banner, and under **Waiting for Your Answer** on any day.
- Once accepted, **Load** puts it in the Diary as before. Undated accepted ones sit under **Suggested by Your Coach** on an empty day.
- A declined one goes away; the coach sees the answer, and a declined workout is never reported as missed. If the coach gives it again, or changes which workout it is, it asks again.
- Answering works offline and goes when the connection returns.

Prescriptions made before this version count as accepted, so nothing already given asks again. An Android app from before it still shows **Load** without asking.

## Notifications

In Settings, Notifications, **Coaching** (shown to people in coaching, all on unless noted), through your Gotify, ntfy or Apprise:

- **New Message**, with **Email Me Too** (off by default; at most one email per conversation every 30 minutes, and it needs an email on your account and email set up on the server).
- **New Recommendation**: a coach asks to coach you, or prescribes a workout.
- **Client's Answer**: someone accepts or declines your request or a workout you prescribed, or leaves.
- For coaches: **Client Completes Prescribed Workout**, **Client Missed a Prescription**, **Client Replied to Feedback**.

They say who, never what was written or the note.

## Privacy

- A coach reads a client's data only while the client has accepted them. Every coach request checks for that pairing, so another coach's client, a request still waiting and coaching that ended are refused alike, admins included.
- The list for adding a client carries only names. A coach's activity feed shows only their own clients.

## Upgrading

Coaching set up before consent stays as it was: a member who already had a trainer keeps them, accepted, with nothing to answer, and their prescriptions, notes, replies and set-video notes all stay. Restoring a backup from before this version does the same.

## Related

- [Programs and templates](programs.md)
- [Diary and set logging](diary.md)
- [Set videos](set-videos.md)
- [Settings reference](settings.md)
