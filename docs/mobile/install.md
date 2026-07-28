# Install the Android app

Each of the three apps publishes a signed Android APK. There is no Play Store listing; grab the APK from GitHub Releases and sideload it. This page covers the install, the first-launch prompt, and the shared-keystore rules that let you upgrade in place without losing data.

## Get the APK

Every release lives on the app's GitHub Releases page. Pick the `.apk` asset that matches your architecture (the standard build is universal, so any device works).

- CookTrace: [`TraceApps/cooktrace/releases/latest`](https://github.com/TraceApps/cooktrace/releases/latest)
- LiftTrace: [`TraceApps/lifttrace/releases/latest`](https://github.com/TraceApps/lifttrace/releases/latest)
- NutriTrace: [`TraceApps/nutritrace/releases/latest`](https://github.com/TraceApps/nutritrace/releases/latest)

Each app also publishes a rolling `dev-latest` pre-release built from the `dev` branch. It is signed with the same keystore as stable, so it upgrades in place. Grab it if you want the leading edge and can live with the occasional rough edge. Occasionally a specific feature milestone gets a numbered `v<version>-dev.N` pre-release for testers who want to pin it. See [Release channels](../reference/release-channels.md) for the full model.

!!! tip "Debug builds"
    The signed release APK enforces HTTPS-only traffic (Android's cleartext-traffic ban). If your server is plain HTTP on the LAN and you cannot set up TLS, build the debug APK yourself with `npm run android:debug`. The debug APK allows `http://` origins. See [HTTPS on the LAN](../self-hosting/lan-https.md) for the four supported paths.

## Enable "Install unknown apps"

Android blocks APK installs from outside the Play Store by default. To allow the install:

1. Open the APK from your file manager or browser.
2. Android prompts you to grant "Install unknown apps" permission to that app (Files, Chrome, whatever opened the APK).
3. Follow the link into Settings, toggle the permission on, and back out.
4. Tap Install.

The permission is per-source, so granting it to your browser does not open the door for other apps.

## First launch

The first time you open the app it runs the setup wizard. Every app asks the same core question up front: **local-only, or connect to a server?**

- **Local-only** leaves the server URL blank. Data lives in on-device SQLite and never leaves the phone. Handy for a single-device workflow or a "try before you host" pass.
- **Connect to my server** takes your server URL and login. The app pulls your account data down, then keeps it in sync in the background.

You can switch later from Settings. See [Local vs server-connected mode](modes.md) for the trade-offs and how the switch handles existing local data.

## Upgrading and downgrading

All three apps are signed with the same TraceApps keystore. That has two useful consequences:

- **Upgrades work in place.** Install a newer APK straight over an older one. Local SQLite, cached images, and preferences all survive.
- **Cross-flavour upgrades work too.** Moving from a debug APK to a release APK (or from `dev-latest` to stable, or vice versa) does not wipe anything, because Android checks the signing key, not the flavour.

!!! danger "Downgrades wipe app data"
    Android refuses to install an APK with a lower `versionCode` than the one already on the device. If you force a downgrade by uninstalling first, the OS deletes the app's private storage on uninstall. That takes your local SQLite database, cached images, and login token with it. If you rely on local mode, [back up first](../self-hosting/backups.md) via Settings before uninstalling anything.

## Related

- [Local vs server-connected mode](modes.md)
- [How sync works](sync.md)
- [First-run wizard](../getting-started/first-run.md)
- [Troubleshooting](../reference/troubleshooting.md)
