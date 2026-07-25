# Statistics and PRs

The Statistics tab is the training-history dashboard. Six views (Overview, Exercise Progress, Records, Volume, Frequency, Body Weight), a shared date range across all of them, and a metric-pill row along the top for switching. Warm-ups are excluded everywhere; only completed working sets with positive weight count.

## The metric-pill layout

Six pills at the top of the page. Tap one to switch view; the range selector below (`1W / 1M / 3M / 6M / 1Y / All`) applies globally so a range you pick on Volume carries over to Frequency and Records without a second click. Scroll position is remembered per metric, so hopping Volume > Frequency > Volume lands back where you were.

- **Overview** the summary board: current streak, longest streak, total workouts, weekly-goal progress, muscle-recovery heatmap, and a body map colored by recent effective sets per muscle.
- **Exercise Progress** pick an exercise; get a weight-per-session sparkline plus per-session sets, volume, and average RPE.
- **Records** every exercise's current top weight and current top estimated 1RM.
- **Volume** weekly total volume bar chart across the range.
- **Frequency** weekly workout count plus a weekday distribution.
- **Body Weight** logged body-weight points over time (sourced from Diary body-stats).

The `All` range resolves the earliest workout in the log via `GET /api/stats/earliest-workout-date` instead of a fixed 10-year window, so imported historical data from Strong or Hevy is visible without a wall.

## Charts

Chart rendering is Chart.js. Two chart types appear across the views:

- **Line / sparkline** weight or estimated-1RM progression per exercise, plus body-weight trend.
- **Bar** weekly total volume, weekly workout count, weekday distribution.

The default chart type for muscle-group breakdowns is user-configurable in Settings > Statistics (bar or line). Two other Settings > Statistics toggles apply globally: **Lock Y-axis to zero** for honest comparisons across ranges, and **Show trend line** for a linear regression on top of the raw points.

## Personal-record detection

A record is set the moment a completed working set beats the previous best for that exercise. Two bests are tracked per exercise:

- **Top weight** the heaviest single set logged.
- **Top estimated 1RM** using the Epley formula (`weight * (1 + reps / 30)`, or the raw weight for 1-rep sets).

Warm-up sets never count. Sets with zero or negative weight never count. Incomplete sets (the completion checkbox never got checked) never count. Detection happens server-side on every stats query, so a PR you set in one session shows up on the Records view before you leave the page.

Inline in the Diary, a set row also gets a PR badge the moment its weight or estimated 1RM exceeds the prior best for that exercise (see [Diary and set logging](diary.md#anatomy-of-a-set)).

## Volume math

Volume for a set is `weight * reps`, adjusted by the exercise's load type: bilateral sets count as one, unilateral sets (single-arm rows, split squats) sum both sides. Warm-ups are skipped. Full logic is in [`server/lib/volume.js`](https://github.com/TraceApps/lifttrace/blob/main/server/lib/volume.js).

The weekly volume view rolls up by ISO week (Monday-anchored). The muscle-group breakdown maps each exercise to its primary muscle tag and sums; secondary muscles are ignored to keep the totals from double-counting a bench press against both chest and triceps.

## Filters and ranges

The range selector (`1W / 1M / 3M / 6M / 1Y / All`) applies to every view. The Exercise Progress view adds a picker for which exercise to plot. The muscle-group breakdown is chart-driven; tap a muscle bar to filter the underlying exercises. There is no dedicated muscle-group filter chip because the body map on Overview covers the same intent visually.

## Streaks and frequency

Streaks come from `GET /api/stats/streaks`: current streak in days, longest streak, and total workouts. A "streak day" is any calendar date with at least one completed set logged. Rest days do not break the streak; only a full day with no logged working set does. The 4-week frequency and weekly goal live on the Overview tile alongside streaks and roll off the current week's ISO Monday.

## Related

- [Diary and set logging](diary.md)
- [Programs and templates](programs.md)
- [Body stats](diary.md#anatomy-of-a-set)
