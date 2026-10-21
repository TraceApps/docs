# Local vs server-connected mode

Every TraceApps Android build can run in one of two modes. You pick one at first launch and can switch later from Settings. This page covers what each mode does, how the switch handles existing data, and what you gain by wiring the app to a self-hosted server.

## The first-launch choice

The welcome screen offers two cards:

- **Use Locally.** All data lives in on-device SQLite (via the Capacitor SQLite plugin). There is no server, no account, and no network traffic beyond what a given feature explicitly needs (Trace AI calls, if configured; Open Food Facts and USDA food lookups on NutriTrace; etc.).
- **Connect to Server.** Enter the URL of your self-hosted instance and sign in. The app talks to that server over HTTPS (NutriTrace 1.5.0 and later also over plain HTTP on your own network, see [Plain-HTTP LAN](../getting-started/lan-http.md#the-android-apps)), keeps a local SQLite mirror as an offline cache, and syncs both directions in the background.

The choice is not permanent. Both modes are first-class code paths; the welcome screen just gets you into one of them cleanly.

## The Sync Options dialog

When you connect from Settings > Server Connection and the phone already holds data of its own (settings count too), the app asks what to do with it. It asks whether or not the server has data. The dialog shows what's on the phone, then offers three choices:

- **Upload Phone to Server.** Sends the phone's data to the account. It's added next to what the server already has, as new records: the server's records stay, nothing is compared by date, and a record both sides have ends up twice. What the phone's copy does replace on the server:
    - **Settings**, in every app: each setting the phone sends replaces the server's.
    - **NutriTrace:** a day's Diary is merged item by item (the same item keeps its newer version), but the phone's body stats and note for a day replace the server's, and so do its Health Connect values and workouts. A food whose barcode the server already has keeps the server's food. Other foods, meals and recipes are added, so ones the server also has appear twice.
    - **LiftTrace:** each workout goes up as its own session, next to any the server has that day. Body stats for a date take the phone's values, and the phone's active program becomes the active one. Programs, templates, custom exercises and cardio are added, so ones with the same name as the server's appear twice. Progress photos and Trace chat stay on the phone.
    - **CookTrace:** recipe and pantry categories are matched by name, keeping the server's. A pantry item the server already has by name keeps the server's. Recipes (with their history), the cook diary and shopping items are added. Cookbooks, custom units, comments and Trace chat go up with the first sync after connecting.
    - **NoteTrace:** notes, labels, checklist items and attachments go up with the first sync after connecting (the summary counts them before then). A label the server also has appears twice; a checklist item both sides have keeps the newer one. Note history stays on the phone.

    Running the upload again after it failed partway sends nothing twice.

- **Download Server to Phone.** Deletes the phone's data and loads the account's instead. CookTrace and NoteTrace also delete the phone's settings. Everything that was only on the phone is gone.
- **Merge Both.** The same as Upload Phone to Server, after which the normal sync brings the server's data down. It's not a last-write-wins merge: records both sides have end up twice.

Which to pick:

- The server already has what's on the phone (you used it connected before, or restored a backup there): **Download Server to Phone**.
- The account is new or empty: **Upload Phone to Server**.
- Both sides have data you want to keep: **Upload Phone to Server** (Merge Both does the same), then delete what's doubled.

!!! warning "There's no undo"
    The choice applies at once, and nothing previews what the server will end up with. If you are unsure, take a [backup](../self-hosting/backups.md) first: full-backup ZIP on the server, or the local backup export from the app's Settings.

## Switching later

Both directions are available from Settings.

- **Local to server:** Settings > Server Connection: enter the server's address, then sign in with a password or one of the server's SSO buttons (SSO here from each app's next release; earlier versions offered only a password). Sync Options appears if the phone holds data, the same either way.
- **Server to local:** Settings > Server Connection > Disconnect. The local SQLite mirror stays on the device, and the app switches to the on-device code path. LiftTrace first sends changes still waiting (see below); the others upload nothing. Nothing is deleted.

!!! note "LiftTrace 1.4.0 and later"
    - Upload sends each logged workout as its own session, plus body stats, cardio, programs, exercises and settings, and the counts it shows first are what it sends.
    - First-run setup asks too, after a password sign-in (not after SSO), with **Upload to Server**, **Replace with Server**, **Merge Both** (which also syncs right away) and **Skip**, which uploads and deletes nothing.
    - **Download Server to Phone** in Settings asks first about changes that never reached a server, never dropping them silently. **Replace with Server** in setup deletes the phone's data right away.
    - **Disconnect** sends changes still waiting first, and asks before dropping any it can't send. Everything else the account had stays on the phone: connecting back to the same account puts it back in place, sending only what changed meanwhile.
    - Signing in as another account shows that account's data, fetched fresh. If the first account left changes that haven't gone up, you're asked first: cancel to keep them for that account, or sign in anyway and they're discarded.

!!! note "CookTrace from its next release"
    - Signing in on the same phone, the app checks whose data it holds before showing anything. Signing back in to the same account keeps everything; another account starts from its own data. If the last account left changes that never reached the server, you're asked before they're discarded.
    - The same server at another address (its LAN address and its domain) is still the same account; where the app can't tell, it asks. Settings, and which units are shown, follow the account too.
    - **Disconnect** keeps your settings on the phone and no longer shows the welcome screen.
    - Connecting to a server that isn't set up yet, the app says to open the server in a browser and create the first account first, instead of saying the server can't be reached.

Reconnecting to the account you disconnected from: LiftTrace and NoteTrace recognize it and send only what's new or changed (in NoteTrace that includes settings: only those changed meanwhile go up). NutriTrace and CookTrace send everything again as new records, so pick **Download Server to Phone** there when the server already has the phone's data. Connecting to a *different* account, pick Upload or Download deliberately: the app has nothing to compare against.

## What each mode gets you

| Capability                                     | Local-only | Server-connected |
| ---------------------------------------------- | :--------: | :--------------: |
| All core features (log, plan, view, export)    |     Yes    |        Yes       |
| Works offline                                  |     Yes    |        Yes       |
| Data never leaves the device                   |     Yes    |         No       |
| Multi-device sync (phone + laptop + tablet)    |     No     |        Yes       |
| Browser access via the web PWA                 |     No     |        Yes       |
| Shared server backups (full-backup ZIP)        |     No     |        Yes       |
| OIDC / SSO login                               |     No     |        Yes       |
| NutriTrace federation (Trace across all apps)  |     No     |        Yes       |
| Cross-user sharing (foods, recipes, cookbooks) |     No     |        Yes       |
| Server-side push notifications (Apprise/ntfy)  |     No     |        Yes       |
| Local device backup (share sheet ZIP/JSON)     |     Yes    |        Yes       |

Local mode is genuinely complete for a single-device workflow. If you never want a server, you never need one. Server mode is the right call as soon as you want the same data on more than one surface, want a browser tab open on your desk, or want federation between the apps.

!!! tip "CookTrace can import from a local NutriTrace"
    CookTrace's first-run wizard offers to import an existing on-device NutriTrace database when both apps run in local mode on the same phone. Handy if you already had NT installed and want to seed pantry entries and food catalog.

## Related

- [Install the Android app](install.md)
- [How sync works](sync.md)
- [First-run wizard](../getting-started/first-run.md)
- [Backups and restore](../self-hosting/backups.md)
