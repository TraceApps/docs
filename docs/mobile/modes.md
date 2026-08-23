# Local vs server-connected mode

Every TraceApps Android build can run in one of two modes. You pick one at first launch and can switch later from Settings. This page covers what each mode does, how the switch handles existing data, and what you gain by wiring the app to a self-hosted server.

## The first-launch choice

The setup wizard offers two options up front:

- **Local-only.** Leave the server URL blank. All data lives in on-device SQLite (via the Capacitor SQLite plugin). There is no server, no account, and no network traffic beyond what a given feature explicitly needs (Trace AI calls, if configured; Open Food Facts and USDA food lookups on NutriTrace; etc.).
- **Connect to my server.** Enter the URL of your self-hosted instance and log in. The app talks to that server over HTTPS, keeps a local SQLite mirror as an offline cache, and syncs both directions in the background.

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

- **Local to server:** Settings > Server Connection > Connect. The merge dialog triggers if the account already has server rows.
- **Server to local:** Settings > Server Connection > Disconnect. The local SQLite mirror stays on the device, and the app switches to the on-device code path. Nothing is uploaded or deleted.

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

Local mode is genuinely complete for a single-device workflow. If you never want a server, you never need one. Server mode is the right call as soon as you want the same data on more than one surface, want a browser tab open on your desk, or want federation between the three apps.

!!! tip "CookTrace can import from a local NutriTrace"
    CookTrace's first-run wizard offers to import an existing on-device NutriTrace database when both apps run in local mode on the same phone. Handy if you already had NT installed and want to seed pantry entries and food catalog.

## Related

- [Install the Android app](install.md)
- [How sync works](sync.md)
- [First-run wizard](../getting-started/first-run.md)
- [Backups and restore](../self-hosting/backups.md)
