# Release channels

The three TraceApps (CookTrace, LiftTrace, NutriTrace) publish across three parallel channels. Same model for all of them.

## Stable

Regular releases follow strict `MAJOR.MINOR.PATCH` semver. Every stable release:

- Lives on the app's GitHub Releases page marked "Latest" (for example `v1.2.3`).
- Publishes multi-arch Docker images tagged `:X.Y.Z`, `:X.Y`, `:X`, and `:latest`.
- Uploads a signed Android APK to the release assets.
- Bumps `package.json`, `src/lib/version.js`, Android `versionCode` + `versionName`, and adds a `CHANGELOG.md` block.

Stable is what non-tester users install. If a tag is not stable, it will not be marked "Latest" on GitHub.

## `dev-latest` (rolling)

`dev-latest` is a floating GitHub pre-release. Every dev-worthy build overwrites the same tag with a fresh APK; no version bump, no CHANGELOG entry, no release notes. It is the primary channel for testers who want "always the newest thing."

The `:dev` Docker tag is the equivalent on the container side. It auto-publishes on every push to the `dev` branch. `dev-latest` and `:dev` refresh in step.

Silent overwrite is the whole point. Doc fixes, small refactors, incremental improvements, and iteration on in-flight features all land here without any release-notes ceremony. If a tester's install upgrade path is "redownload the APK and reinstall," they get the newest thing every time.

## Milestone `v<version>-devNN`

Numbered dev pre-releases exist for feature milestones testers should be able to pin, install intentionally, or reference by name in bug reports. Examples: a new AI capability, a new wearable integration, a big backup change.

Each numbered milestone gets:

- A permanent GitHub pre-release at `v<version>-devNN` with tester-facing notes.
- A `## [X.Y.Z-devNN] - <date>` block in `CHANGELOG.md`, capturing the delta since the previous milestone or the last stable.
- A version bump in `package.json` and `src/lib/version.js`. Android `versionCode` and `versionName` are NOT bumped on dev iterations.
- A Docker tag `ghcr.io/traceapps/<app>:X.Y.Z-devNN` alongside `:dev`.
- A refresh of `dev-latest` to point at the same commit.

The `<version>` reflects what will land as the next stable release. After NT `v1.0.3`, the next milestone dev build is `v1.0.4-dev01` (patch-worthy) or `v1.1.0-dev01` (minor-worthy).

### Iteration number format

- Zero-padded two digits for 1 through 9: `dev01`, `dev02`, …, `dev09`.
- Natural two digits from 10 onward: `dev10`, `dev11`, …, `dev99`.
- No dot between `dev` and the number. This makes the whole `devNN` one alphanumeric semver pre-release identifier, which stays inside SemVer 2.0.0 §9 (which forbids leading zeros in *pure-numeric* identifiers like `.01`). It also sorts identically in lex and semver order, so the GitHub Tags page, `gh release list`, Docker Hub tag lists, and the in-app updater all agree on the order `dev01 < dev02 < … < dev09 < dev10 < … < dev99 < <stable>`.
- Not expected to hit `dev100+` in any patch cycle; if that ever happens, revisit padding width.
- Historical note: NT tags `v1.1.0-dev.1` through `v1.1.0-dev.15` used the older dotted format. They sort correctly ahead of new no-dot tags (`dev.15 < dev16`), so no retroactive rename was needed.

## How Docker tags map

| Tag | Channel | Updates when |
|-----|---------|--------------|
| `:X.Y.Z` (e.g. `1.2.3`) | Stable | Never; immutable pin |
| `:X.Y` (e.g. `1.2`) | Stable | Any `1.2.z` patch release |
| `:X` (e.g. `1`) | Stable | Any `1.y.z` minor or patch |
| `:latest` | Stable | Every stable release |
| `:X.Y.Z-devNN` | Milestone dev | Never; immutable pin to that milestone |
| `:dev` | Rolling dev | Every push to the `dev` branch |

Legacy `:X.Y.Z-rc.N` tags from before the semver switch remain in the registry; no new `-rc` tags are cut.

## How Android APKs map

All APKs, stable and dev, are signed with the shared TraceApps keystore. That has two consequences:

- **Upgrades work in place.** A newer APK installs straight over an older one. Local SQLite, cached images, and preferences all survive.
- **Cross-channel upgrades work too.** Moving from stable to `dev-latest`, from a numbered milestone to stable, or from `dev-latest` to a milestone does not wipe anything, because Android checks the signing key, not the channel.

Downgrades still fail. Android refuses an APK with a lower `versionCode`, and a forced downgrade via uninstall wipes the app's private storage. Back up first via Settings if you plan to move backwards.

## How to pick

- **Most testers:** install `dev-latest` and let it float. Redownload from the same URL whenever you want the newest build.
- **Reporting a bug:** if the maintainer asked you to try a specific milestone, or you want a stable label to reference, install a numbered `v<version>-devNN`.
- **Everyone else:** install the latest stable release. It is what non-tester users are running.

## Related

- [Docker image tag matrix](image-tags.md)
- [Changelogs](changelogs.md)
- [Install the Android app](../mobile/install.md)
- [Updating](../getting-started/updating.md)
