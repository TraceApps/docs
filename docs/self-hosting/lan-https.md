# HTTPS on the LAN

The three apps default to secure cookies, and the Android release APK enforces the platform cleartext-traffic policy. Plain `http://192.168.x.y:3001` (or `:3002` / `:3003`) will silently fail: cookies get dropped, the WebView refuses to load, and login appears to "just not work". You have four supported ways to give a LAN-only install real HTTPS. Pick one.

If you genuinely need plain HTTP inside your LAN, see [LAN HTTP setup](../getting-started/lan-http.md) for the escape hatches (`INSECURE_COOKIES=1` and a debug-build APK). Everything below assumes you want proper HTTPS.

## Path 1: real domain + Let's Encrypt

The simplest and most Android-friendly option: point a real DNS name at your box (a subdomain like `cook.example.com` is fine) and let Caddy or nginx terminate TLS with an ACME cert. Cookies stay `Secure`, both PWA and Android see valid CA-signed certs, and there is nothing to install on the client.

Caddy makes this a two-line block:

```caddy
cook.example.com {
    reverse_proxy cooktrace:3001
}
```

Point your router's public IP at the box or use split-horizon DNS if you want the domain to resolve only inside the LAN. Even with LAN-only resolution, Let's Encrypt's HTTP-01 challenge needs port 80 briefly reachable from the public internet, or use the DNS-01 challenge instead (Cloudflare, Route53, and most self-hosted DNS providers work out of the box with Caddy plugins). See [Reverse proxy](../getting-started/reverse-proxy.md) for the full snippets and equivalent nginx / Traefik configs.

## Path 2: Cloudflare Tunnel or Tailscale

If you don't want to open any port on your router, expose the app through a tunnel service that terminates TLS elsewhere. The app itself keeps talking plain HTTP inside the trusted network; the client always sees HTTPS.

**Cloudflare Tunnel.** Install `cloudflared`, log in, and add a public hostname pointing at `http://cooktrace:3001`. Cloudflare's edge holds the cert, and the tunnel origin gets a valid `Origin` header. Beware the free-tier 100 MB proxied-body cap: it can bite full-backup restore. Split large restores over the LAN instead.

**Tailscale.** Enable HTTPS in the Tailscale admin console, then run `tailscale serve https / http://localhost:3003` on the host (adjust the port for the app you're exposing: NT `3001`, LT `3002`, CT `3003`). Every machine on your tailnet reaches `https://<host>.<tailnet>.ts.net` with a valid cert issued by Tailscale's ACME. The Android app installed on any tailnet-connected phone works with no extra config.

In both cases, cookies stay `Secure`, the Android release APK is happy, and you never issue self-signed anything.

## Path 3: self-signed CA on-device

You can run your own tiny CA (mkcert, step-ca, smallstep, or a hand-rolled OpenSSL script), issue a cert for `cook.lan` or `192.168.1.10`, and terminate TLS at Caddy or nginx.

Every client that talks to the server has to trust your CA. On desktops this is usually fine; on Android it's the sticking point.

!!! warning "Android certificate store gotcha"
    Android release APKs (including the TraceApps ones from GH Releases) only trust the **system** CA store by default. A cert installed via Settings, Security, Encryption and credentials, Install a certificate lands in the **user** store, which release-built apps ignore unless a Network Security Configuration explicitly opts in. You either need to add the CA to the system store (root/Magisk trick, or a custom ROM), sideload the debug APK (which trusts the user store), or move to Path 1 or Path 2.

For desktop PWA installs this path works well: import the CA once into Firefox / Chrome / the OS keychain and everything after is transparent. Just do not expect the release Android app to pick it up.

## Path 4: sideload the debug APK

The debug APK is built with `android:usesCleartextTraffic="true"` in the manifest **and** trusts the user CA store. If your install is genuinely LAN-only and you accept the tradeoff, grabbing the debug APK from the GH Releases pre-release channel (`dev-latest`) lets you talk to `http://192.168.x.y:3001` (or whichever host port the app is on) or an HTTPS endpoint using a user-installed self-signed cert.

Downsides: no Play-Store update path, and it is a debug build, so any additional security hardening the release variant enables is absent. Handy for testing, less good for daily driving. On the server side, remember to also set `INSECURE_COOKIES=1` if you go plain-HTTP, so the auth cookie is not dropped by the WebView.

## What actually breaks without HTTPS

Two things, both silent-ish:

1. **The auth cookie is `Secure` by default.** Browsers and WebViews drop it on `http://`. First request after login succeeds; every subsequent one is 401. Setting `INSECURE_COOKIES=1` removes the `Secure` attribute; do this only inside a trusted network.
2. **Android release APK cleartext policy.** The manifest is `usesCleartextTraffic="false"` in the release variant. Plain-HTTP requests just refuse. There is no in-app toggle to override this; you either need HTTPS on the server or the debug APK.

## Related

- [Reverse proxy](../getting-started/reverse-proxy.md)
- [LAN HTTP setup](../getting-started/lan-http.md)
- [Cloudflare Tunnel and Tailscale recipes](tunnels.md)
