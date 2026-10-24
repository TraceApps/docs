# Cook Diary and Meal Planner

Both live on the same page (`/diary`). The distinction is a single column on each row: `kind = 'cooked'` for a log entry, `kind = 'planned'` for something scheduled. That means the same UI, the same drag-and-drop, and one-tap promotion from planned to cooked.

## List view

Rows grouped by date, 60 days back and 30 days forward by default. Each row shows the recipe hero, name, meal type, per-cook rating, notes, and a status pill (Cooked / Planned). Planned rows have a **Mark as Cooked** button that flips `kind` and stamps the entry to today (or keeps the planned date if you'd rather).

Add a cook or a plan from the **+** button: pick a recipe, date, and meal type (Breakfast / Lunch / Dinner / Snack, or unset). Notes and a per-cook rating are optional.

**Servings** are optional and never filled in: a plan has none unless you set them, with **Set Servings** (it starts on the recipe's own number; the close button takes it off again). With none, nothing shows on the plan and [Add from Planned Cooks](shopping.md#add-from-planned-cooks) buys the recipe as written; with some, the Diary shows them on the plan and the list scales the recipe to them. **Plan a Cook** on a recipe card, and **Plan** on a recipe, open the same dialog for that recipe, without leaving the page.

## Week view

**Week**, next to List, Month and Photos, shows one week at a time, cooked and planned together. The arrows move a week back or forward, and tapping the dates returns to this week. Weeks start on Sunday or Monday, as your device's locale has them.

- **Days behind you** show what you cooked: the photo, a Cooked badge and your rating.
- **Today and the days ahead** show your planned cooks. One with servings set has a stepper, and the shopping list scales the recipe to it; to set or clear them, use **Edit Plan**. A plan for today (or an earlier day still waiting) has **Mark as Cooked**.
- **A free day** has a **Plan a Cook** button that opens the planner on that date.
- **Any Day This Week**: a cook planned with **Any Day That Week** waits in a tray above the days. Tap it and choose **Pick a Day** to give it one; marking it cooked dates it today when today is in its week. The week's shopping list includes it either way.
- Tap a cook you made to open the recipe **as you made it**, at the version it was made from, with what has changed since (see [History](recipes.md#history)). Long-press any cook for As You Made It, Open Recipe, Mark as Cooked and Delete.
- **Move or edit a plan.** Long-press a planned cook (in any view) for **Move to Another Day**, a calendar from today on, and **Edit Plan**, for its day (or any day that week), meal and servings. With a mouse or trackpad you can also drag a planned cook onto another day in the Week view. A cook you've made stays where it was.
- **The shopping list follows.** When the list already holds a planned cook's ingredients, editing or deleting the cook, or marking it cooked, changes the list to match straight away, and the message says so: more servings add to what's on the list, a deleted cook takes its items off, and a cooked one takes off what's left of its share. What you've bought and cleared, or deleted from the list, isn't put back; building the week's list again adds everything it needs. A cook moved into another week is counted once when that week's list is built.
- With a [household](settings.md#household-per-user) set up, a planned cook warns about an allergen or diet that's a problem for someone home that day ("Gluten · Sam" on a weekday, nothing on the weekend Sam is away).

For the current week and weeks ahead, a summary above the days counts the meals planned, the ingredients to buy, the ones already in the pantry, and the items about to expire that the plan uses up. **Build Shopping List** puts the week's plan on the list (see [Add from Planned Cooks](shopping.md#add-from-planned-cooks)); once the list holds it, the button reads **Update Shopping List**.

### Uses Up

Below the days, **Uses Up** lists what's in stock and about to expire (by the week's end, and at least a week ahead), with the days each has left and which planned cook uses it. Under it come the recipes you haven't planned that would use the most of those items ("Uses Avocados and Cilantro"), then the ones that share the most with what's planned, so one pack of tortillas covers two dinners; **Plan** opens the planner with the recipe picked, on the week's next free day. The summary's Uses Up tile jumps here.

CookTrace doesn't know how much is left in a package, so Uses Up goes by use-by dates rather than leftovers.

### Build My Week

**Build My Week**, in the week's summary, picks dinners for you:

- **Dinners** (starts at the number of free days left, up to five) and **Servings**.
- **Use Expiring** favors recipes that use what's about to expire, **Not Cooked Lately** passes over what you cooked in the last week or two, and **Under 30 Min** keeps to quick recipes.

The Servings row sets every dinner's servings; left alone, they have none. With a [household](settings.md#household-per-user) set up, a recipe that's a problem for someone home that day is left out of that day, and one with an ingredient someone home doesn't like ranks lower and says so. A note says how many recipes were left out; **Include Them** brings them back, with their warnings showing.

The picks are ranked by how much of each recipe is in your pantry, what it uses up, ingredients it shares with the other picks (one bunch of cilantro for two dinners), and how long since you cooked it. Recipes already planned that week aren't picked. Each says why ("Uses Avocados and Cilantro", "6 of 7 on hand", "Not cooked in 5 weeks"), and **Swap** brings the next best for that slot. The totals at the bottom (dinners, things to buy, expiring items used) update as you go. **Add to Week** plans them as dinners on the week's free days from today, in order; any more than there are free days go to Any Day This Week.

Trace can do the same in a chat ("plan three dinners for next week"): its planning tool takes servings when you say how many, and can plan for any day of a week.

## Month calendar

Toggle to Month for a month grid with recipes as chips inside each cell. Its weeks start on your locale's first day of the week (Monday in most of Europe, Sunday in the US), as the Week view's do; before the next release it always started on Sunday. Drag a chip between cells to reschedule. Drag between planned and today's cell to "cook now". The calendar respects the same `meal_type` coloring and shows a small stack when a day has more than three entries (tap to expand).

## Plan-then-cook workflow

The typical week:

1. Open the Diary's **Week** view and move to next week.
2. Tap **Plan a Cook** on the days you want, or plan a few for **Any Day That Week**, and set the servings if you want to cook for a different number than the recipe's. With a household set up, the recipe list warns about what's a problem for them.
3. Tap **Build Shopping List** in the week's summary: every planned recipe's ingredients go on the list, scaled to each cook's servings and added up across recipes. Building it again after changing the plan updates the list rather than doubling it.
4. During the week, tap **Mark as Cooked** as you go.

Planned entries that lapse without being marked cooked stay planned; delete or drag them to a new date if you want to keep the calendar honest.

## Heatmap

The top of the Diary page shows a contribution-graph-style **heatmap** of cook density over the last year. Colors pull from your accent color with luminance steps by cook count. Click a cell to jump to that day's row in the list. Backed by `GET /api/cook-diary/heatmap`.

## Per-recipe cook history

Every recipe view shows its own cook history as a table (date, meal type, rating, notes). Handy when you want to remember "did I use the small pan or the big pan last time?" and the note you left past-you is right there. Each cook links to the version of the recipe it was made from, and the List view opens a cook the same way.

## Meal-plan-to-shopping-list

**Add from Planned Cooks** on the Shopping page (`POST /api/shopping/from-plan`):

- Walks every planned entry in the chosen date range, scaling each recipe to the entry's servings.
- Adds up the same ingredient across recipes and across units of one kind (cups with milliliters, grams with kilograms).
- Skips anything stocked in the pantry (via each ingredient's pantry link), and anything already checked off on the list.
- Keeps, on each list item, the recipes and planned cooks it's for, so building the same range again replaces their share instead of adding it twice, and a cook taken off the plan takes its share out.

`POST /api/shopping/from-recipe/:id` handles a single recipe ("Add to Shopping List" on a recipe card), scaled with `?servings=` when given.

See [Shopping](shopping.md#add-from-planned-cooks) for the details.

## Data model

`cook_diary` carries: `user_id`, `recipe_id` (nullable, in case the recipe is deleted later), `date` (`YYYY-MM-DD`), `kind` (`cooked | planned`), `servings`, `meal_type`, `rating`, `notes`, `any_day` (a plan for any day of the week its `date` starts), timestamps. Deletes are soft (`deleted_at`), which keeps `last_cooked_at` roll-ups on the recipe honest during sync.

## Related

- [Recipes](recipes.md)
- [Pantry](pantry.md)
- [Feature tour: Shop from the plan](features.md#shop)
