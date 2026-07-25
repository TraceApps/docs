# Radio player

The Radio tab is a real music player, not a background afterthought. It exists because lifting to music matters and because reaching for a second app breaks flow. Two source types: your own music server, or public internet radio streams.

## Sources

### Self-hosted music servers

Configure once in Settings > Integrations > Radio > Self-Hosted Music. Pick a provider, enter server URL, credentials, done. Every provider then browses inside LiftTrace: albums, artists, playlists, search, queue.

- **Subsonic API** compatible: Navidrome, Airsonic, Funkwhale, Gonic. Server URL + username + password. Endpoints proxy under `/api/subsonic/rest/*`.
- **Jellyfin**. Server URL + username + password. Endpoints proxy under `/api/subsonic/provider/jf/*`.
- **Plex**. Server URL + Plex token (not password). Endpoints proxy under `/api/subsonic/provider/plex/*`.
- **Emby**. Server URL + username + API key. Endpoints proxy under `/api/subsonic/provider/emby/*`.

Server-side proxy means your credentials never round-trip through the browser and CORS never gets in the way. It also means the LiftTrace container needs to be able to reach your music server, which is usually a same-network non-issue.

### Internet radio streams

Streaming stations get their own page inside Radio. Add / edit / delete / group stations. Supported protocols:

- **Icecast** (`http(s)://...`)
- **Shoutcast** (`http(s)://...`)
- **HLS** (`.m3u8`)

Now-playing metadata is extracted from `StreamTitle` in the Icy stream, the `text=` query param some Shoutcast servers use, RDS Italia frames, and embedded ID3 frames. The extractor lives in `src/lib/radio-icy.js`.

An **Icon suggest** endpoint (`GET /api/radio-proxy/icon-suggest`) grabs a station's favicon or logo when you add it. You can override with a custom URL per station.

## SSRF safety: `ALLOW_PRIVATE_RADIO_URLS`

The radio proxy fetches remote stream URLs from the LiftTrace container. To prevent Server-Side Request Forgery, it blocks loopback, RFC1918, and IPv6 ULA ranges by default. If a station URL resolves to `127.0.0.1`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, or `fc00::/7`, the request is refused with:

```text
Private / loopback addresses are blocked. Set ALLOW_PRIVATE_RADIO_URLS=1 to enable LAN streams.
```

If you actually stream from an Icecast box on your LAN and you understand the trade-off, opt in:

```env
ALLOW_PRIVATE_RADIO_URLS=1
```

Accepted values: `1` or `true`. Anything else keeps the guard on. The startup log emits a warning when the opt-out is active. Do not enable this on a public-facing deployment unless you have thought carefully about what a malicious station URL could reach on your internal network.

HLS bypasses the proxy entirely: `hls.js` in the browser does its own XHR for the manifest and segments, and the radio proxy does not rewrite HLS manifests. Same-origin manifests work; cross-origin needs CORS on the origin server.

## Web playback: the MSE pipeline

Web playback runs a single `<audio>` element with a single `SourceBuffer` and appends decoded chunks sequentially, matching `timestampOffset` across tracks so audio never pauses between them. Full detail in `src/lib/mseStreamer.js`.

The reason is not audiophile bragging rights. Chrome's tab-freeze heuristic on locked screens will suspend an `<audio>` element that goes quiet, even briefly, breaking auto-advance. Keeping the audio graph unbroken across track boundaries keeps LiftTrace's tab exempt from the freeze so the queue keeps advancing when the phone is in your pocket.

Because MSE keeps the source open across tracks, the `ended` event never fires, and end-of-queue detection has to be explicit inside the streamer. If you see weird pause-at-end behavior on the web, that is where to look.

## Android: unified Media3 ExoPlayer

The Android app routes both radio and library through Media3 ExoPlayer with a single `MediaSession`, backed by `RadioPlayerPlugin.java` and `RadioPlaybackService.java`. One session, one lockscreen notification, one Bluetooth headset. The lockscreen layout swaps at runtime:

- **Radio (streaming station)**: single `[Stop]` button.
- **Library (queued tracks)**: `[Prev | Next | Stop]`.

Layout switch happens via `setCustomLayout` plus `setLibraryLayout(boolean)` from JavaScript.

The notification is posted **manually** through a custom `Notification.Builder` and `startForeground`, not through Media3's default `DefaultMediaNotificationProvider`. This is intentional: LiftTrace needs the radio-vs-library layout swap and the mixed-mode session's custom actions, and the default provider does not give enough control for both.

Do not "simplify" this back to the default provider. The custom builder is the whole reason the lockscreen behaves correctly across the two modes.

## Playback niceties

- **Keep original format**: bit-perfect toggle in Settings > Integrations > Radio. Falls back to 320 kbps MP3 transcoding when the queue mixes codecs.
- **2-track pre-fetch**: skip is instant on library queues.
- **Crossfade**: 1 to 12 seconds, configurable in Settings.
- **Lock-screen controls**: Media Session API on the web (play, pause, previous, next), MediaSession on Android with hardware key support (headset play/pause, Bluetooth previous/next).
- **Web Audio tap**: powers the Trace FAB frequency visualizer without adding a second audio graph.

## Mini-player and full player

The mini-bar sits at the bottom of every screen when audio is playing. Tap it to expand into the full player: album art, seekable progress bar, shuffle, repeat, queue. Drag the mini-bar to dismiss it (audio keeps playing; the bar reappears when a new track starts or when you reopen the Radio tab).

## Related

- [Environment reference](../self-hosting/env-vars.md)
- [Trace in LiftTrace](trace.md)
- [Diary and set logging](diary.md)
