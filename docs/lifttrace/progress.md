# Progress photos

Dated photos kept alongside your body stats, with a drag-to-compare
before/after view for any two dates. The scale tells you a number changed;
this is the part that shows what actually changed.

Photos live on their own page at `/progress`. It is a real, bookmarkable
route but deliberately not a nav-bar tab, since this is content you open
every few weeks rather than every session.

## Getting there

- **Statistics > Body Weight** has a "Progress photos" card at the top.
- Straight to `/progress` if you have it bookmarked.
- The Body Stats sheet in Diary links across after you add a photo.

## Adding a photo

Two places, both writing to the same timeline:

- **Diary > Body Stats sheet.** An "Add photo" button sits under the
  measurements. This attaches to whichever date the Diary is showing, so
  it is the one to use when backdating.
- **The Progress page itself.** The toolbar button (and the button in the
  empty state, first time round) always attaches to today.

Images only, 20 MB each. The file is validated by its actual bytes rather
than its name or the browser's claimed type, and SVG is refused outright.

There is no limit on photos per day, so a front and a side shot on the
same date both show up.

### iPhone photos

iPhones shoot HEIC by default, which Chrome, Firefox and Android WebView
cannot display at all. LiftTrace converts HEIC to JPEG in the browser
before uploading, so a photo taken on an iPhone still renders everywhere
you later look at it. The converter is only fetched when you actually
pick a HEIC file, so it costs nothing otherwise.

## Browsing

Newest first, grouped by month, covering the last year. Each photo shows
its date and, when you logged a weight that day, the weight next to it,
so the number and the picture sit together without a second lookup.

One column on a phone, filling out to four across on a desktop.

Tap any photo to open it full screen.

## Comparing

- **Compare first and latest** (appears once you have three or more
  photos) opens your oldest and newest side by side in one tap. For most
  people this is the comparison they actually came for.
- **Compare two** turns the grid into a picker; tap any two photos.

The comparison stacks both photos in a single frame and wipes between
them with a draggable divider, rather than shrinking each into half the
screen. Drag with a finger, a mouse, or a pen. The arrow keys nudge the
divider (hold `Shift` for larger steps, `Home` and `End` jump to either
extreme). The older photo's date is stamped bottom-left, the newer one
bottom-right.

## Deleting

Each photo has a delete button, behind a confirmation. Unlike the app's
other uploads, this removes the file from disk as well as the database
row, so a deleted photo does not sit in `uploads/` forever.

The same applies to deleting your account or using Settings > Clear my
data: the image files go too, not just the records.

## Storage, sync, and backup

- Photos are stored under `UPLOADS_PATH/body-stats/` with randomized
  filenames.
- They sync across devices through the normal sync path, deletions
  included.
- Full backups contain both the database rows and the image files, and a
  restore brings back both.

**Photos never expire.** That is deliberate: a progress timeline whose
early entries disappear is not a progress timeline. This is the opposite
of what a short coaching clip would want, which is why video review is
tracked as its own separate feature rather than sharing this one's rules.

!!! note "Who can read your photos"
    Uploaded files are served without authentication, protected only by
    unguessable filenames. This is long-standing LiftTrace behavior that
    progress photos inherit rather than introduce: the Android app cannot
    attach an auth header to an `<img>` tag, so avatars and exercise media
    work the same way. Progress photos are more personal than an avatar,
    so it is worth knowing. Anyone who has the exact URL can open it;
    nobody can list or guess their way to it. If that tradeoff does not
    suit you, keep the instance off the public internet (a VPN or
    Tailscale) rather than relying on the filename.

## Automation

All optional, each behind the flag it already uses:

- **Webhook** `progress_photo.logged` fires whenever a photo is attached,
  no matter which of the three surfaces did it. Subscribe in
  Settings > Webhooks with `WEBHOOKS_ENABLED=1`.
- **MCP** `list_progress_photos` (read) and `add_progress_photo` (write),
  under the existing `mcp:read` / `mcp:write` scopes. See
  [Model Context Protocol](mcp.md).
- **REST** `GET /api/v1/body-stats/photos?start=&end=` and
  `POST /api/v1/body-stats/photos`.

Both the MCP tool and the REST endpoint take an image **URL**, not file
bytes, since neither surface handles multipart uploads. They are for
attaching something you already host; uploading from a device goes
through the app.

## Related

- [Statistics & PRs](statistics.md)
- [Diary & set logging](diary.md)
- [Model Context Protocol (MCP)](mcp.md)
