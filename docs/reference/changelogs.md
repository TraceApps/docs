# Changelogs

Each app maintains its own `CHANGELOG.md` in-tree rather than duplicating them here. Follow the source of truth per app:

- **CookTrace:** [`TraceApps/cooktrace/CHANGELOG.md`](https://github.com/TraceApps/cooktrace/blob/main/CHANGELOG.md)
- **LiftTrace:** [`TraceApps/lifttrace/CHANGELOG.md`](https://github.com/TraceApps/lifttrace/blob/main/CHANGELOG.md)
- **NutriTrace:** [`TraceApps/nutritrace/CHANGELOG.md`](https://github.com/TraceApps/nutritrace/blob/main/CHANGELOG.md)

## Release cadence

Everything lands on the `dev` branch first. Pushes to `dev` publish the `:dev` Docker tag automatically (see [Docker image tag matrix](image-tags.md)) but the changelog does not move; `dev` is a rolling working copy.

Releases are cut by squash-merging `dev` into `main` with a single version-bump commit that updates `package.json`, `version.js`, the Android `build.gradle` (`versionCode` and `versionName`), and `CHANGELOG.md` together. That commit is then tagged (`v1.2.3`), which triggers the Docker workflow to publish the `1.2.3`, `1.2`, `1`, and `latest` tags in one build.

## Downloadable APKs

The Android APK is uploaded to the tagged GitHub Release for each version. Grab it from the Releases page:

- [CookTrace releases](https://github.com/TraceApps/cooktrace/releases)
- [LiftTrace releases](https://github.com/TraceApps/lifttrace/releases)
- [NutriTrace releases](https://github.com/TraceApps/nutritrace/releases)

A `dev-latest` pre-release APK exists on each Releases page for testers who want to run the same code that the `:dev` Docker tag serves. It is manually promoted (not auto-published on every `dev` push), so treat it as "the maintainer's latest hand-checked build" rather than continuous.

## Related

- [Docker image tag matrix](image-tags.md)
- [Updating](../getting-started/updating.md)
- [Install the Android app](../mobile/install.md)
