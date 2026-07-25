# Open Food Facts

Open Food Facts is the primary source of ready-made barcode and food lookups in NutriTrace. It requires no API key, no account, and no configuration to use in the default cloud mode. For air-gapped setups or households with flaky connectivity, NutriTrace also supports a **local Parquet mirror** that pulls the whole OFF product database down to disk and serves lookups from there.

## Cloud usage (default)

Nothing to set up. Open Food Facts is enabled out of the box under **Settings → Connected Services → Open Food Facts**. Barcode scans and name searches hit `world.openfoodfacts.org` directly. Search language and country biasing are configurable per user (`offSearchLang`, `offSearchCountry`), and a per-food import preference (`offImportBasis`) picks per-serving vs per-100g when both are published by OFF.

**Optional OFF account credentials** enable the **Share to Open Food Facts** button in the food editor. When a product is not yet in OFF, the button uploads your entry with its picture; when the product already exists, the button flips to **View on OFF** and opens the wiki page so you can suggest edits through OFF's own moderation and history workflow. Credentials live under `offUsername` / `offPassword`, are user-scoped, and are refused entirely when the instance is in air-gap mode (see below).

## Local mirror (Parquet or DuckDB)

For air-gapped hosts, laggy WAN, or just faster lookups, NutriTrace can point at a local snapshot of the OFF database.

**Enable it** by setting one environment variable at the container level:

```env
OFF_LOCAL_DB=/data/off-mirror/off.parquet
```

The path is inside the container. Point it at a bind-mounted directory that survives restarts. The default docker-compose file mounts `${OFF_LOCAL_DB_HOST_PATH:-./off-mirror}` at `/data/off-mirror`, so the value above works out of the box.

**File format.** The mirror is either a **Parquet file** (the current shape, downloaded from Hugging Face) or a **DuckDB native file** (the older shape). NutriTrace detects the shape from the file itself: Parquet is opened as an in-memory DuckDB view over the file; native `.duckdb` files are opened directly in read-only mode. Existing installs with a pre-rc.39 `.duckdb` file continue to work; new installs default to Parquet.

**Auto-download.** If `OFF_LOCAL_DB` points at a path that does not yet exist, NutriTrace kicks a background download at startup. The default download source is:

```
https://huggingface.co/datasets/openfoodfacts/product-database/resolve/main/food.parquet?download=true
```

Override with `OFF_LOCAL_URL` if you host your own snapshot or need a specific mirror. The download is ~7-8 GB; give it time on a first boot. The lookup path falls through to the public OFF API during the download unless air-gap mode is set.

**Refresh.** The mirror is a snapshot, not a live view. **Settings → Connected Services → Open Food Facts → Auto-refresh schedule** (admin-only) lets you set how often to re-download. **Refresh Now** kicks a refresh on demand. Refresh writes to a `.new` sibling file and atomic-renames on completion, so a failed download never leaves you with a half-written mirror.

!!! warning "Bind-mount must be a directory, not the file"
    If `OFF_LOCAL_DB` points at a path that Docker created as a file bind-mount (as opposed to a directory bind-mount), Docker's auto-mount behaviour creates a directory at the mount point and refresh breaks with `EISDIR`. Mount the parent directory (`/data/off-mirror`) as a directory bind-mount and let NutriTrace manage the file inside it.

## Air-gap mode

Set `OFF_LOCAL_ONLY=1` and NutriTrace refuses to fall back to the public OFF API for anything. Barcode misses and name-search misses return "not found" rather than round-tripping to the internet. The proxy endpoint that normally passes OFF lookups through to the public API refuses with `503` and a message about the air-gap policy (`server/routes/proxy.js`).

Air-gap mode also disables the OFF account contribute path entirely, since Share-to-OFF requires reaching out to `world.openfoodfacts.org` to POST. The button hides in that mode.

The full local-mode env-var summary:

| Var | Purpose |
|---|---|
| `OFF_LOCAL_DB` | In-container path to the Parquet or DuckDB file. Presence enables the mirror. |
| `OFF_LOCAL_ONLY` | `1` to refuse the public OFF API entirely (air-gap). |
| `OFF_LOCAL_URL` | Override the download source. Defaults to the Hugging Face `food.parquet`. |
| `OFF_LOCAL_DB_HOST_PATH` | Compose-only var used as the host side of the bind-mount default. |

## Fallback matrix

| State | Barcode lookup | Name search | Share-to-OFF |
|---|---|---|---|
| Cloud only (default) | OFF cloud | OFF cloud | Available if creds set |
| Mirror configured | Local first, cloud on miss | Local first, cloud on miss | Available if creds set |
| Mirror + `OFF_LOCAL_ONLY=1` | Local only | Local only | Refused |

## Not backed up

The OFF mirror is **excluded from full backups** (`DEPLOY.md:218`). On a fresh restore the file re-downloads on the configured schedule; if you want an off-site snapshot of the mirror itself, use rsync, restic, or borg against the host bind-mount path. Excluding it from backups is deliberate: 7-8 GB of publicly-mirrorable data does not need to sit inside every nightly ZIP.

## Licensing

!!! warning "ODbL, the Open Database License"
    Open Food Facts data is released under the **Open Database License (ODbL)**, not AGPL. If you run a multi-user NutriTrace instance with the OFF local mirror enabled, an ODbL disclosure banner surfaces in the admin UI (new in 1.0.3, per `LICENSES.md`). Bulk redistribution of the mirror or of OFF-sourced data beyond your own users has to comply with ODbL's share-alike terms. Personal and internal use is unrestricted.

The ODbL banner shows only in the admin view; end users don't see it. It is a one-time reminder rather than a nag.

## Related

- [Foods, meals & recipes](foods.md)
- [USDA & Mealie integration](usda-mealie.md)
- [Barcode & Scan Label](scan.md)
- [Environment reference](../self-hosting/env-vars.md)
