# Coaching

!!! note "Coming in 1.5"
    Coaching is on the dev branch and arrives with NutriTrace 1.5. The `dev-latest` builds have it now (see [Release channels](../reference/release-channels.md)).

A coach helps a client with their nutrition: they see the client's days, recommend targets, day plans, meals and recipes, and the two talk in one conversation. The coach only suggests. Nothing in the client's targets, diary or catalog changes until the client accepts.

Coaching needs a server with more than one account (multi-user mode). It works in the browser and in the Android app connected to that server, with or without a connection once it is set up. Anyone not in coaching sees NutriTrace exactly as before: no extra tabs, requests or timers.

## For admins

- **Give the Coach role.** In **Settings → Users**, set an account's role to **Coach**. A Coach is an ordinary user everywhere else. Admins can coach too, with no role change.
- **Invite with a coach.** An invite can name a coach. The new account gets that coach's request at its first sign-in, and still has to accept it.
- **See and end pairings.** Settings → Users lists every coach and client pairing, and you can end any of them. Admins manage roles and pairings but see a client's data only when they are that client's coach.
- **Nothing to configure.** Coaching uses the server you run: no environment variable, port or extra service. Notifications go through each person's own push service; the optional email copy uses the server's SMTP settings. Message photos are kept under `UPLOADS_PATH/coaching` and are never served publicly, so back up `UPLOADS_PATH` as usual. Full backups include pairings, recommendations, messages and photos.

## For coaches

Goals has a **Clients** tab for anyone with the Coach role (and for an admin who coaches).

### Asking someone to be your client

**Add Client** lists the server's accounts by name. Pick one and add an optional note. They see your request on their Coach tab and accept or decline it. You see nothing of theirs until they accept, and asking reveals nothing: a request is never refused because the person already has a coach. A person accepts one coach at a time.

### What you see

Open a client to see:

- **Overview.** Their last 7 days against their targets (calories and macros, on target or not), and their weight trend.
- **Diary.** Any of their days, read-only.
- **Plans.** Day plans you send them (below).
- **Targets.** Their targets now, their Adaptive TDEE and weight trend for context, and the targets you recommend.
- **Conversation.** Your messages and recommendations.

You never see their wellness data (Fitbit, Garmin, Withings, Health Connect).

### Recommending targets

The Targets editor starts from the client's targets. Change calories, protein, carbs or fat (grams or percent of calories, the same every day or by weekday), or recommend a calorie goal mode, and add a note. A newer recommendation replaces one still waiting for an answer.

### Day plans

On **Plans**, pick a date and fill the client's own meals (their meal names) with items from their foods, meals and recipes, your own, or an Open Food Facts search. The builder shows the day's totals against their targets. Each item is a copy, so editing the food later doesn't change a plan already sent. **Copy to Other Days** sends the same plan for each day you pick. One plan per day: sending again replaces the plan on that day. You see whether each plan was accepted, declined or expired, and which meals were logged.

### Send to Client

In the meal editor, **Send to Client** sends one of your meals or recipes to a client with a note. It goes as a copy (with its ingredients), so it works with sharing off and your later edits don't reach it. On Accept the client gets their own copy.

### Trace for coaches

Ask Trace to look at a client and draft targets or a day plan ("draft a lighter plan for Jordan tomorrow"). Trace reads the client's summary and foods and prepares a draft; a card offers **Open in Targets** or **Open in Plans**. The editor opens with the draft so you can change anything and send it yourself. Trace never sends a recommendation, and never reads the conversation.

### Ending coaching

**End Coaching** on the client's page. You lose access to their data and the conversation; recommendations still waiting are withdrawn.

## For clients

Goals has a **Coach** tab when you have a coach or a request.

- **Requests.** "Sam wants to be your coach" says what the coach would see: your diary, targets, weight and your foods, meals and recipes. It also says that if the coach asks their Trace about you, what Trace reads goes to the coach's AI provider. Accept or decline. Accepting is your consent; until then the coach sees nothing.
- **Recommendations.** Each arrives as a card on the Coach tab and in the conversation; targets also show on Goals (and once on the Diary), and a day plan on its Diary day. Accept or decline, with a note if you like.
    - **Targets.** The card shows each target now, recommended and the change. Accepting first saves your targets as a template ("Before coach change" and the date), then changes only the recommended ones. **Undo** puts them back. With Dynamic or Adaptive calorie goals, the card says Accept switches you to Fixed.
    - **Day plans.** After Accept, **Log This** logs a meal and **Log All** the rest; tapping an item opens the usual add sheet with the plan's amount. Logged items are ordinary diary items. A food you don't have is copied into your foods when you log it. Deleting an item from the Diary brings Log This back for it. A plan you didn't answer shows Expired after its day.
    - **Meals and recipes.** Accepting adds your own copy to Meals or Recipes, marked "From Your Coach", with its ingredients' foods.
- **Trace for clients.** Ask Trace to explain a recommendation ("why does my coach want more protein?"). Trace reads the recommendation itself, never the conversation.
- **Leave Coach** on the Coach tab ends it. You keep the conversation to read.

## The conversation

One conversation per coach and client, on the client's Coach tab and the coach's Conversation tab: text up to 4,000 characters and one photo per message. Photos are private to the two of you. Read receipts, unread counts on the tabs, and a dot on Goals in the bottom bar and the sidebar for unread messages and anything waiting for your answer. You can delete your own messages. Trace never reads the conversation.

The conversation checks for new messages when you open it and every 10 seconds while it is on screen, and not otherwise.

## Notifications

Under **Settings → Notifications → Coaching** (shown only to people in coaching):

- **New Message** (on by default): a message from your coach or client.
- **New Recommendation** (on by default): a recommendation or a coaching request.
- **Client's Answer** (coaches, on by default): a client accepted or declined, with what it was (targets, the plan's date, or the meal's name).
- **Email Me Too** (off by default): also email new messages, at most one per conversation every 30 minutes.

They go through your ntfy, Gotify or Apprise. A message notification says who wrote, never what.

## Privacy

- Nothing is shared before the client accepts the request, and a coach sees only what the request said: diary, targets, weight trend, foods, meals and recipes. Never wellness data.
- Every coach view of a client checks for an active pairing between that coach and that client, on every request.
- Ending coaching (either side, or an admin) removes the coach's access at once. Deleting an account removes its pairings, recommendations, messages and photos.
- Trace never reads the conversation. What a coach's Trace reads of a client (their summary and foods, to draft a recommendation) goes to the AI provider the coach set up, like everything else Trace reads for them.

## Offline and Android

- **Messages** written with no connection wait on the device and go once when the connection returns (each has its own id, so a retry never sends twice).
- **Answers** work offline: accepting targets changes them on the device at once; accepting a day plan lets you log it; accepting a meal or recipe brings your copy when the connection returns. The answer goes then too.
- **Logging a plan** works offline, as the Diary does.
- **A coach's views** opened before stay readable offline. Sending a recommendation, asking, accepting a request and ending coaching need a connection.
- **Android** keeps the conversation and recommendations on the phone and syncs them with the server. With no server (Android local mode), the Coach and Clients tabs say coaching needs a server.

## Sharing

The food, meal and recipe editors have a **Share** button, shown when an admin has turned sharing on in **Settings → Sharing**: the same Private, Everyone or Specific People choice as Bulk Share, for one item. It needs a connection. Coaching does not need sharing: plans and sent meals carry their own copy.
