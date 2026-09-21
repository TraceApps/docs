# Set videos

A short clip attached to one set, so a coach can look at the lift, or you
can look at it yourself a week later. The note half of coaching already
existed; this gives it something to point at.

## Filming a set

From the Diary, open an exercise's `...` menu and pick **Film a set**.
Choose which set the clip belongs to (it defaults to the next one you have
not finished), then either:

- **Record.** Films in the app, up to 60 seconds, with a live preview and a
  countdown. This is the option to prefer: recording here uses a modest
  bitrate, so a set lands around 10 to 20 MB instead of the 150 MB a phone
  camera produces at its default settings.
- **Choose a clip.** Anything already on the phone, up to 200 MB. Nothing is
  re-encoded on the server, so a long clip at full phone quality can hit
  that ceiling. If it does, the app says so rather than failing silently.

Attaching a clip needs a connection. A video is far too large to hold in
the offline queue the way a progress photo is, so the sheet says as much
when you are offline. Everything else in the Diary still works without one.

## Watching one back

A filmed set is marked twice, so a collapsed card never hides the fact that
footage exists:

- A small play badge on the set row itself.
- A play chip on the exercise header, with a count when more than one set
  on that exercise was filmed.

Either opens the player, which names what you are watching ("Back Squat,
set 1"), plays the clip, and shows the coach's note underneath if there is
one.

## The coach's side

In **Coaching**, open a member and then one of their sessions. Any exercise
they filmed carries a **Watch** chip. Opening it gives the coach the same
player plus a note composer:

1. Play the clip and pause where the problem is.
2. Tap the stamp button to capture that moment.
3. Write the note and save.

The note is stored against the exercise the way coach notes always were, so
it also appears in the member's normal feedback surfaces. What the clip adds
is the timestamp: in the member's player the note carries a chip like
`0:04`, and tapping it seeks the video straight there.

The member can reply from the player without leaving it, and the coach sees
that reply next time they open the clip. A coach cannot attach or delete
clips, only watch and comment.

## Who can see a clip

Footage of someone lifting in their gym is treated as more private than an
avatar or a shared exercise demo:

- Clips are **never served from `/uploads`**. That tree is mounted ahead of
  the auth check so images can load without a header, which makes anything
  in it readable by anyone with the URL. Clips are read through
  `GET /api/set-media/:id/file`, which checks the row's owner.
- Only **the owner and that owner's assigned trainer** can open one. An
  instance admin is not an exception, matching progress photos.
- Deleting a clip removes the row and the file. The coach's note survives,
  minus its pointer to the video.

## Storage and clean-up

Nothing expires on its own. Deleting someone's training footage on a
schedule they did not choose is worse than the disk it costs, and on a
self-hosted instance the disk is yours to manage.

So the numbers are visible instead. **Settings > Workout > Set Videos**
shows how many clips you have, what they occupy, and the date of the oldest
one, with two clear-outs: everything older than a month, or older than six
months. Both delete the rows and the files, and keep any coach notes.

Clips are included in full backups (rows and files), sync between your own
devices as rows (the video itself is fetched on demand), and are removed
along with your data if you delete your account or clear your data.

## Limits

| | |
|---|---|
| Max size | 200 MB per clip |
| In-app recording | 60 seconds |
| Formats | MP4, WebM, QuickTime, validated by the file's actual bytes |
| Clips per set | No limit |
| Server-side transcoding | None |

## Related

- [Coaching](coaching.md) for the trainer and member roles this builds on.
- [Progress photos](progress.md), which established the private-media
  handling clips reuse.
