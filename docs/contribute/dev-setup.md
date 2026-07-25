# Dev setup (all three)

The three apps share a stack, so this page covers all of them. Where the setup diverges (backend port, native scraper, DuckDB), it is called out per app.

## Prerequisites

- **Node 20** (the Docker base image is `node:20-alpine` for CookTrace and LiftTrace, `node:20-slim` for NutriTrace). Older Node majors will occasionally build but are not tested. nvm is the easy path:

    ```bash
    nvm install 20
    nvm use 20
    ```

- **Git**, plus a GitHub account if you plan to open a PR.
- **Python 3** on your host if you want CookTrace's recipe-scrapers tier to run outside Docker. Not required for LiftTrace or NutriTrace.
- **JDK 17 + Android Studio** only if you plan to build the Android app. Not needed for backend or PWA work.

## Clone the app you want to work on

Each app lives in its own repo under [TraceApps on GitHub](https://github.com/TraceApps). Clone the one you want, then check out the `dev` branch (everything lands on `dev` first; `main` is squash-merged release commits only).

=== "CookTrace"

    ```bash
    git clone https://github.com/TraceApps/cooktrace
    cd cooktrace
    git checkout dev
    npm install
    cd server && npm install && cd ..
    ```

=== "LiftTrace"

    ```bash
    git clone https://github.com/TraceApps/lifttrace
    cd lifttrace
    git checkout dev
    npm install
    cd server && npm install && cd ..
    ```

=== "NutriTrace"

    ```bash
    git clone https://github.com/TraceApps/nutritrace
    cd nutritrace
    git checkout dev
    npm install
    cd server && npm install && cd ..
    ```

## Ports

| App | Vite dev server | Backend Express |
|-----|-----------------|-----------------|
| CookTrace  | `:5175` | `:3001` |
| LiftTrace  | `:5173` | `:3003` |
| NutriTrace | `:5173` | `:3001` |

Vite falls forward if a port is already taken (`:5174`, `:5175`), so if you run two apps side by side, check the terminal for the real URL.

## Two dev flows

Both flows work for all three apps. Pick based on what you are changing.

### Path A: Vite dev server with API proxy

Fast HMR, but the frontend runs on a separate origin (`:5173` or `:5175`) from the backend (`:3001` or `:3003`). Vite proxies `/api` and `/uploads` through to the backend. Handy for iterating on markup and Svelte components.

```bash
# terminal 1: backend
cd server && node index.js

# terminal 2: frontend
npm run dev
```

Open the Vite URL that the second terminal prints.

### Path B: Single-origin build

Build the frontend, drop it into `server/dist`, and let Express serve the built bundle. This matches exactly what the production Docker image does, so it is the reliable path when you want to verify a branch end to end.

```bash
npm run build
rm -rf server/dist && cp -r dist server/dist
cd server && node index.js
```

Open `http://localhost:3001` (or `:3003` on LiftTrace). Re-run all three lines after any frontend change; the server templates `dist/index.html` at startup, so a rebuild needs a server restart to be picked up. Backend-only changes just need the server restarted.

!!! tip "Blank page in Path A"
    A stale PWA service worker will keep serving the previous shell even though your new build is on disk. Open DevTools, unregister the service worker, clear site data, hard-reload. If that does not fix it, switch to Path B; single-origin sidesteps the whole class of problem.

## Per-app quirks

**CookTrace** has a Python tier for recipe scraping (`recipe-scrapers`, 300+ site-specific extractors). The Docker image bakes `python3` and the library in. Locally, set `PYTHON_BIN=python3` (or the full path to a venv) and install the library yourself: `pip install recipe-scrapers`. Without it, imports fall back to schema.org JSON-LD only.

**LiftTrace** builds pre-rendered rest-cue audio tones on first Android build via `npm run build:rest-tones`. It runs automatically as part of `npm run android`, `android:run`, and `android:debug`. If you only run `vite build`, the tones will be missing from the APK; use one of the `android:*` scripts instead.

**NutriTrace** uses `node:20-slim` (Debian) rather than Alpine because DuckDB (Open Food Facts local mirror) needs glibc. Local dev on Alpine or musl-libc distros will hit `Error loading shared library ld-linux-x86-64.so.2` when the OFF mirror layer tries to load. On glibc-based Linux, macOS, and Windows this is a non-issue.

## Android debug builds

Every app has an `android:debug` (LiftTrace) or `android:apk:debug` (CookTrace, NutriTrace) script that produces an unsigned APK at `android/app/build/outputs/apk/debug/app-debug.apk`.

Wireless ADB is the smoothest handset workflow. Enable **Developer options > Wireless debugging** on the phone, pair it once (the rotating port shows in the phone's Wireless debugging panel), then:

```bash
adb connect <phone-lan-ip>:<port>
adb -s <phone-lan-ip>:<port> install -r android/app/build/outputs/apk/debug/app-debug.apk
```

Debug builds are exempt from Android's cleartext-traffic ban, so they can talk to a plain `http://` server on your LAN. Release builds cannot; see [HTTPS on the LAN](../self-hosting/lan-https.md) for the fix.

## Commit and PR conventions

- Short one-line commit messages. Body only for architectural notes.
- Do not add `Fixes #NN` or `Closes #NN` trailers without asking; the maintainer prefers to close issues by hand after a release lands.
- No DCO, no CLA. Just open the PR against `dev`.
- If your change touches user-facing strings, add them to `src/i18n/en.json` in the same commit. See [Translations (i18n)](translations.md).
- If your change adds a Settings-panel field, add matching keywords to `SECTION_KEYWORDS` in `src/routes/Settings.svelte` so the in-Settings search finds it.

## Related

- [Translations (i18n)](translations.md)
- [Trace tool catalog](../reference/trace-tools.md)
- [Docker image tag matrix](../reference/image-tags.md)
