# Feature tour

A tour through the four flows you will use most. Nothing here requires config beyond a working install. Deep dives on each area live in their own pages, linked at the bottom.

## Log today's workout

Open **Diary**. The date bar defaults to today; arrow keys or the calendar icon jump you around. On an empty day you see a row of quick-load cards for recent workouts, one tap to prefill.

Add an exercise from the **+ Exercise** picker at the bottom. Search matches names, aliases (`BB`, `DB`, `BP`, `OHP`, `DL`, `SQ`, `RDL`), and equipment tags. Tap the exercise and it lands in the diary with one blank set row waiting for weight and reps.

Every set row has:

- **Weight** and **reps** inputs. Long-press either to open the Gym Tools plate calculator or the lbs/kg converter.
- A **completion checkbox**. Ticking it triggers the rest timer and unlocks superset gating.
- An optional **RPE** field. Toggle per-set RPE tracking in Settings > Workout.
- A **warm-up** toggle on the row context menu. Warm-up sets do not count toward volume, PRs, rest-timer firing, or the completion summary.

Sets you finish today auto-prefill next time from the last completed working set for that exercise. When the workout is in an active program, the program's prescription wins.

Finish the workout by tapping the summary bar at the bottom. You get a 1080x1350 completion card with your total volume, top sets, and any PRs the app detected. Native share sheet on mobile, download on desktop.

Full details in [Diary and set logging](diary.md).

## Follow an active program

Open **Programs**. Every fresh install seeds a handful of starter templates (Push/Pull/Legs, Upper/Lower, Full Body 3x, plus two branded programs). Pick one, tap **Activate**, and it becomes your active program. Only one program is active at a time.

Back in Diary, the empty day now shows a **Load today's workout** button that prefills the whole session (exercises, target reps, target weight, warm-ups, RPE targets) from the current template.

Multi-week programs (v1.0.1) add a Week tab strip in the Program editor. Each week gets its own Sets / Reps / Tempo / Rest / Load matrix. Advance modes: `sessions` (default; the app moves you forward after each completed workout) or `calendar` (moves by real days). On the last week you choose whether the plan **holds** or **repeats** from week one. You can also nudge the week cursor manually from the Load Workout sheet.

See [Programs and templates](programs.md).

## Check PRs

Open **Statistics**. The metric-pill row across the top switches between:

- **Overview**: streaks, workouts in range, average per week, 90-day activity heatmap.
- **Exercise Progress**: pick an exercise, see top-set weight and estimated 1RM over time.
- **Records**: recent PRs plus PRs grouped by muscle category.
- **Volume**: weekly totals plus muscle-group breakdown so imbalances stand out.
- **Frequency**: weekly counts and weekday distribution.
- **Body Weight**: trend line, min/max, change over range.

The date range (1W / 1M / 3M / 6M / 1Y / All) applies globally. Warm-ups are excluded from every calculation.

PR detection also runs inline in Diary: when a completed set beats the prior top weight or top estimated 1RM for that exercise, the row surfaces a badge, and the completion summary lists it.

## Use the radio during a session

The **Radio** tab is a full music player, not a background afterthought. Two source types:

- **Self-hosted music server**: Subsonic-compatible (Navidrome, Airsonic, Funkwhale, Gonic), Jellyfin, Plex, or Emby. Configure once in Settings > Radio and browse albums, artists, playlists, and search inside LiftTrace.
- **Streaming stations**: Icecast, Shoutcast, or HLS (`.m3u8`) URLs. Add, group, and edit them on the Radio page. Now-playing metadata comes from `StreamTitle`, `text=` params, RDS Italia, or embedded ID3.

The player runs a mini-bar across every screen so you can log sets without leaving the music. On Android, playback goes through Media3 ExoPlayer with a lockscreen notification and Bluetooth headset controls. On the web, an MSE-based pipeline keeps audio flowing between tracks so a locked phone tab does not stall on auto-advance.

See [Radio player](radio.md).

## Talk to Trace

The floating Trace assistant is docked bottom-right on every page. Two things it does that a generic chat window does not:

1. **Live workout context.** Every message carries your active program, the last 14 workouts (warm-ups filtered), your top PRs, a 30-day body-weight trend, streaks, and today's coach prescription if there is one. Ask "how was my push day last Thursday" and Trace already knows.
2. **Hold-to-log.** Press and hold the FAB for 700 ms. It turns red, the face swaps to a mic, and voice recording starts. Speak the workout in plain English (`bench 3 sets of 5 at 225, then 3 sets of curls`), release the FAB, and the parsed sets land in the Smart Add review sheet. Slide off more than 100 px to cancel mid-record.

Provider is your call: Claude, OpenAI, Gemini, or any OpenAI-compatible endpoint including a local Ollama or LM Studio. Set once server-side to lock the config for every user, or let each user bring their own key from Settings > Trace.

See [Trace in LiftTrace](trace.md).

## Related

- [Diary and set logging](diary.md)
- [Programs and templates](programs.md)
- [Exercise library](exercises.md)
- [Radio player](radio.md)
- [Trace in LiftTrace](trace.md)
- [Workout history import](import.md)
