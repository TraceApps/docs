# HTTPS on the LAN

The Trace apps default to secure cookies, and the Android release APK enforces the platform cleartext-traffic policy. Plain `http://192.168.x.y:3001` (or `:3002` / `:3003` / `:3004`) will silently fail: cookies get dropped, the WebView refuses to load, and login appears to "just not work". You have four supported ways to give a LAN-only install real HTTPS. Pick one.

If you genuinely need plain HTTP inside your LAN, see [LAN HTTP setup](../getting-started/lan-http.md) for the escape hatches (`INSECURE_COOKIES=1` and a debug-build APK). Everything below assumes you want proper HTTPS. A self-signed certificate is a supported way to get it, including on the Android release app: see Path 3.

## Path 1: real domain + Let's Encrypt

The simplest and most Android-friendly option: point a real DNS name at your box (a subdomain like `cook.example.com` is fine) and let Caddy or nginx terminate TLS with an ACME cert. Cookies stay `Secure`, both PWA and Android see valid CA-signed certs, and there is nothing to install on the client.

Caddy makes this a two-line block:

```caddy
cook.example.com {
    reverse_proxy cooktrace:3003
}
```

Point your router's public IP at the box or use split-horizon DNS if you want the domain to resolve only inside the LAN. Even with LAN-only resolution, Let's Encrypt's HTTP-01 challenge needs port 80 briefly reachable from the public internet, or use the DNS-01 challenge instead (Cloudflare, Route53, and most self-hosted DNS providers work out of the box with Caddy plugins). See [Reverse proxy](../getting-started/reverse-proxy.md) for the full snippets and equivalent nginx / Traefik configs.

## Path 2: Cloudflare Tunnel or Tailscale

If you don't want to open any port on your router, expose the app through a tunnel service that terminates TLS elsewhere. The app itself keeps talking plain HTTP inside the trusted network; the client always sees HTTPS.

**Cloudflare Tunnel.** Install `cloudflared`, log in, and add a public hostname pointing at `http://cooktrace:3003`. Cloudflare's edge holds the cert, and the tunnel origin gets a valid `Origin` header. Beware the free-tier 100 MB proxied-body cap: it can bite full-backup restore. Split large restores over the LAN instead.

**Tailscale.** Enable HTTPS in the Tailscale admin console, then run `tailscale serve https / http://localhost:3003` on the host (adjust the port for the app you're exposing: NT `3001`, LT `3002`, CT `3003`). Every machine on your tailnet reaches `https://<host>.<tailnet>.ts.net` with a valid cert issued by Tailscale's ACME. The Android app installed on any tailnet-connected phone works with no extra config.

In both cases, cookies stay `Secure`, the Android release APK is happy, and you never issue self-signed anything.

## Path 3: self-signed CA on-device

You can run your own tiny CA (mkcert, step-ca, smallstep, or a hand-rolled OpenSSL script), issue a cert for `cook.lan` or `192.168.1.10`, and terminate TLS at Caddy or nginx.

Every client that talks to the server has to trust your CA. On desktops that is a one-off import, and on Android it is one too.

!!! tip "The release APKs trust your own CA"
    Android apps ignore user-installed CAs by default, which is why this path fails for most apps. The Trace release APKs opt in: their `network_security_config.xml` lists both the **system** and the **user** trust anchors. Install your CA under Settings, Security, Encryption and credentials, Install a certificate, and the release app from GH Releases accepts your cert. No root, no custom ROM, no debug build.

For desktop PWA installs this path works the same way: import the CA once into Firefox, Chrome or the OS keychain and everything after is transparent.

## Path 4: sideload the debug APK

Only for plain `http://`. A self-signed cert needs no debug build any more: see Path 3. The debug APK permits cleartext traffic, so if your install is genuinely LAN-only and you accept the tradeoff, grabbing it from the GH Releases pre-release channel (`dev-latest`) lets you talk to `http://192.168.x.y:3001`, or whichever host port the app is on.

Downsides: no Play-Store update path, and it is a debug build, so any additional security hardening the release variant enables is absent. Handy for testing, less good for daily driving. On the server side, remember to also set `INSECURE_COOKIES=1` if you go plain-HTTP, so the auth cookie is not dropped by the WebView.

## What actually breaks without HTTPS

Two things, both silent-ish:

1. **The auth cookie is `Secure` by default.** Browsers and WebViews drop it on `http://`. First request after login succeeds; every subsequent one is 401. Setting `INSECURE_COOKIES=1` removes the `Secure` attribute; do this only inside a trusted network.
2. **Android release APK cleartext policy.** The release variant ships `network_security_config.xml` with `cleartextTrafficPermitted="false"`, which the platform enforces over the manifest's `usesCleartextTraffic` attribute. Plain-HTTP requests just refuse. There is no in-app toggle to override this; you either need HTTPS on the server or the debug APK. Certificates are a different question: a self-signed one works on the release build once its CA is installed on the phone (Path 3).

## Related

- [Reverse proxy](../getting-started/reverse-proxy.md)
- [LAN HTTP setup](../getting-started/lan-http.md)
- [Cloudflare Tunnel and Tailscale recipes](tunnels.md)
