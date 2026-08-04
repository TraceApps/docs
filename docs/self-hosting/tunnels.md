# Cloudflare Tunnel and Tailscale

Two ways to give a self-hosted TraceApps container a real HTTPS URL without opening a port on your router or running your own ACME. Both terminate TLS outside your network, so the container itself keeps talking plain HTTP on its private address.

Both fit the same shape: run a small daemon next to the app that dials out to the tunnel provider, register a hostname, done. No inbound port-forwarding, no cert renewal to babysit, no dynamic DNS. Cookies come back `Secure`, so the Android release APK is happy, and the PWA installs cleanly.

## Cloudflare Tunnel

Install `cloudflared`, log in with `cloudflared tunnel login` (opens a browser to authorise against a Cloudflare Zero Trust team), and create a named tunnel:

```bash
cloudflared tunnel create cooktrace
cloudflared tunnel route dns cooktrace cook.example.com
```

Then a config file at `~/.cloudflared/config.yml`:

```yaml
tunnel: cooktrace
credentials-file: /root/.cloudflared/<tunnel-uuid>.json

ingress:
  - hostname: cook.example.com
    service: http://cooktrace:3001
  - service: http_status:404
```

Run it as a systemd service (`cloudflared service install`) or as a compose sidecar. Cloudflare's edge terminates TLS with a Cloudflare-issued cert; the tunnel origin serves plain HTTP on the docker network.

You do not need your own domain. `cloudflared tunnel --url http://cooktrace:3001` prints a throwaway `https://<random>.trycloudflare.com` URL that lasts for the life of the process. Good for a quick share, not for daily use (URL changes on every restart, no auth in front).

!!! warning "100 MB proxied-body cap on the free plan"
    Cloudflare's free plan caps proxied request bodies at 100 MB. A full-backup restore ZIP that pushes past that limit gets a `413 Payload Too Large` at the edge, before the app ever sees it. Same for very large recipe-import ZIPs in CookTrace. Two workarounds: temporarily move the restore ZIP into `BACKUPS_PATH` on the host and pick it from the in-app list (skips the upload entirely), or restore over the LAN.

Everything else the app does (photos, backups written locally, the settings-sync stream) fits comfortably under the cap.

## Tailscale

Tailscale keeps the app entirely private (only devices on your tailnet can reach it) and issues a valid `*.<tailnet>.ts.net` cert from its own ACME.

Install Tailscale on the host, join your tailnet, then enable HTTPS in the admin console (Settings, DNS, "Enable HTTPS"). Point it at the app:

```bash
tailscale serve --bg https / http://localhost:3001
```

Adjust the port for the app you're exposing: NT `3001`, LT `3002`, CT `3003`.

Every tailnet-connected device reaches `https://<host>.<tailnet>.ts.net` with a real cert. The Android app installed on any tailnet phone works with no extra config. Certs renew automatically.

If you want the app reachable from outside your tailnet without opening a router port, use `tailscale funnel` instead of `serve`. Funnel exposes the tailnet URL to the public internet through Tailscale's edge, with the same cert. Requires enabling Funnel in the admin console and accepting the ACL.

Tailscale does not proxy request bodies through an edge, so there is no 100 MB cap to worry about; a full-backup restore streams the whole way through.

## Which one

- **Tailscale** if you host for yourself, your household, and a few friends who don't mind installing Tailscale. Least public exposure, no request-size trap.
- **Cloudflare Tunnel** if you need a stable public URL anyone can reach without account setup, and you can live with the 100 MB body cap.

Both compose fine with any of the proxies in [Reverse-proxy recipes](proxy-recipes.md) if you want a proxy in front, for example to add rate-limiting or an auth gate. In that case terminate TLS at the tunnel and speak plain HTTP to Caddy or nginx inside your network.

## Related

- [Reverse-proxy recipes](proxy-recipes.md)
- [HTTPS on the LAN](lan-https.md)
- [Environment reference](env-vars.md)
