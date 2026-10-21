# Local vs server-connected mode

Every TraceApps Android build can run in one of two modes. You pick one at first launch and can switch later from Settings. This page covers what each mode does, how the switch handles existing data, and what you gain by wiring the app to a self-hosted server.

## The first-launch choice

The setup wizard offers two options up front:

- **Local-only.** Leave the server URL blank. All data lives in on-device SQLite (via the Capacitor SQLite plugin). There is no server, no account, and no network traffic beyond what a given feature explicitly needs (Trace AI calls, if configured; Open Food Facts and USDA food lookups on NutriTrace; etc.).
- **Connect to my server.** Enter the URL of your self-hosted instance and log in. The app talks to that server over HTTPS (NutriTrace 1.5.0 and later also over plain HTTP on your own network, see [Plain-HTTP LAN](../getting-started/lan-http.md#the-android-apps)), keeps a local SQLite mirror as an offline cache, and syncs both directions in the background.

The choice is not permanent. Both modes are first-class code paths; the wizard just gets you into one of them cleanly.

## The merge dialog

If the account you log into has server data **and** you already have local data on the device, the app cannot silently pick one. It shows a merge dialog with three options:

- **Upload local.** Push what is on the device up to the server. Server rows for the same records get overwritten by the local ones.
- **Download server.** Pull server data down and replace the local copy. Local-only entries the server has never seen are discarded.
- **Merge (last-write-wins).** Combine both sides. Rows that exist on both sides are resolved by `updated_at`; the newer one wins. Rows unique to either side are kept.

Merge is the safe default when you have been logging on both surfaces (say, phone in local mode and the web PWA on your laptop). Upload / Download are the right calls when one side is clearly authoritative and you want a clean state.

!!! warning "Merge is not a dry-run"
    The three options apply immediately. There is no preview. If you are unsure which side is authoritative, take a [backup](../self-hosting/backups.md) first: full-backup ZIP on the server, or the local backup export from the app's Settings.

## Switching later

Both directions are available from Settings.

- **Local to server:** Settings > Server Connection: enter the server's address, then sign in with a password or one of the server's SSO buttons (SSO here from each app's next release; earlier versions offered only a password). The merge dialog triggers if the account already has server rows, the same either way.
- **Server to local:** Settings > Server Connection > Disconnect. The local SQLite mirror stays on the device, and the app switches to the on-device code path. Nothing is uploaded or deleted.

!!! note "LiftTrace 1.4.0 and later"
    - The dialog is **Sync Options**: **Upload Phone to Server**, **Download Server to Phone** or **Merge Both**. Upload sends each logged workout as its own session, plus body stats, cardio, programs, exercises and settings, and the counts it shows first are what it sends. Progress photos and Trace chat stay on the phone. Uploading again sends nothing twice.
    - **Download Server to Phone**, and **Replace with Server** in setup, delete the phone's data as they say. Changes that never reached a server are asked about first, never dropped silently.
    - **Disconnect** sends changes still waiting first, and asks before dropping any it can't send. Everything else the account had stays on the phone: connecting back to the same account puts it back in place, sending only what changed meanwhile.
    - Signing in as another account shows that account's data, fetched fresh. If the first account left changes that haven't gone up, you're asked first: cancel to keep them for that account, or sign in anyway and they're discarded.

!!! note "CookTrace from its next release"
    - Signing in on the same phone, the app checks whose data it holds before showing anything. Signing back in to the same account keeps everything; another account starts from its own data. If the last account left changes that never reached the server, you're asked before they're discarded.
    - The same server at another address (its LAN address and its domain) is still the same account; where the app can't tell, it asks. Settings, and which units are shown, follow the account too.
    - **Disconnect** keeps your settings on the phone and no longer shows the welcome screen.
    - Connecting to a server that isn't set up yet, the app says to open the server in a browser and create the first account first, instead of saying the server can't be reached.

Switching back and forth is safe as long as you understand the merge dialog. The one thing to avoid is switching to a *different* server account without picking Upload or Download deliberately; a same-account reconnect uses `updated_at` correctly, but a fresh account has nothing to compare against.

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
