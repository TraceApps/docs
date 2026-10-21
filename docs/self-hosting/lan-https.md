# HTTPS on the LAN

The Trace apps default to secure cookies, and the Android release APK enforces the platform cleartext-traffic policy. Plain `http://192.168.x.y:3001` (or `:3002` / `:3003` / `:3004`) will silently fail: browsers drop the sign-in cookie, so signing in loops back to the sign-in page, and the Android release apps refuse to connect. The exception is NutriTrace's Android app from 1.5.0: it connects to a server on your own network over plain HTTP after a warning (see [Path 4](#path-4-plain-http-on-your-own-network-nutritrace)). For HTTPS, you have three supported ways to give a LAN-only install a certificate. Pick one.

If you genuinely need plain HTTP inside your LAN, see [LAN HTTP setup](../getting-started/lan-http.md) for the escape hatches (`INSECURE_COOKIES=1` for browsers, and NutriTrace's Android app on your own network). Everything below assumes you want proper HTTPS. A self-signed certificate is a supported way to get it, including on the Android release app: see Path 3.

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

## Path 4: plain HTTP on your own network (NutriTrace)

NutriTrace's Android app, from 1.5.0, connects over plain `http://` to a server on your own network: a private address such as `http://192.168.1.20:3001`, a Tailscale `100.x` address, or a local name such as `http://nutritrace:3001` or `http://nutritrace.local:3001`. Before the first connection it asks **Connect Without Encryption?**: the connection isn't encrypted, and anyone on the same network can read and change what's sent, including your password. Choose **Connect Anyway** only on your home network or over a VPN. The answer is kept for that server, and Settings, Server Connection shows **Not Encrypted**. A plain-HTTP address on the public internet is refused before anything is sent. Details in [LAN HTTP setup](../getting-started/lan-http.md#the-android-apps).

CookTrace, LiftTrace and NoteTrace still need HTTPS on Android. Every APK on GitHub Releases, the `dev-latest` pre-release included, is a signed release build, so none of them talks plain HTTP. Only a debug build you make yourself from the source does.

## What actually breaks without HTTPS

Two things, both silent-ish:

1. **The sign-in cookie is `Secure` by default.** Browsers drop it on any plain `http://` address other than `localhost`, so signing in loops back to the sign-in page. NutriTrace 1.4.0, NoteTrace 1.1.0 and LiftTrace 1.4.0 and later say so on the sign-in and setup screens; CookTrace from its next release. Setting `INSECURE_COOKIES=1` removes the `Secure` attribute; do this only inside a trusted network. The Android app signs in with a token and isn't affected by it.
2. **Android release APK cleartext policy.** The CookTrace, LiftTrace and NoteTrace release APKs include `network_security_config.xml` with `cleartextTrafficPermitted="false"`, which the platform enforces over the manifest's `usesCleartextTraffic` attribute. Plain-HTTP requests just refuse, with no in-app way around it, so they need HTTPS. NutriTrace's app from 1.5.0 allows cleartext at the platform level and checks each address itself: plain HTTP only to a server on your own network, after the warning (Path 4). Certificates are a different question: a self-signed one works on every release build once its CA is installed on the phone (Path 3).

## Related

- [Reverse proxy](../getting-started/reverse-proxy.md)
- [LAN HTTP setup](../getting-started/lan-http.md)
- [Cloudflare Tunnel and Tailscale recipes](tunnels.md)
