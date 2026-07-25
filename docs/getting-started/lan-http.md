# Plain-HTTP LAN (INSECURE_COOKIES)

This is the first-time gotcha almost every self-hoster hits. Symptoms: the app loads over `http://192.168.x.x:3000`, you fill in login, the page reloads, and you're right back at the login screen with no error. Nothing in the logs looks wrong.

The cause is cookies, not credentials.

## Why cookies fail over plain HTTP

By default the app issues its session cookie with the `Secure` attribute set, which tells the browser: "only send this cookie back over HTTPS." Reach the app over plain `http://`, and the browser silently drops the cookie on the floor. The next request has no session, so the server bounces you back to the login screen. From the outside it looks like a bad password.

This is the browser doing exactly what it's supposed to. The default is `Secure: true` because that's the safe choice for anything reachable from outside a trusted network.

## The escape hatch: `INSECURE_COOKIES=1`

Set the `INSECURE_COOKIES` env var to `1` and the server issues cookies without the `Secure` attribute. The browser then keeps them across plain-HTTP requests and login works.

=== "CookTrace"

    ```yaml
    services:
      cooktrace:
        environment:
          INSECURE_COOKIES: "1"
    ```

=== "LiftTrace"

    ```yaml
    services:
      lifttrace:
        environment:
          INSECURE_COOKIES: "1"
    ```

=== "NutriTrace"

    ```yaml
    services:
      nutritrace:
        environment:
          INSECURE_COOKIES: "1"
    ```

Then `docker compose up -d` to recreate the container.

!!! danger "Only for trusted LAN, never for internet-exposed installs"
    `INSECURE_COOKIES=1` strips the single browser-side defense that keeps your session token off open WiFi. Use it only when the app is reachable exclusively from a network you control (your home LAN, a Tailscale mesh, a VPN). Never on anything that resolves from the public internet, even briefly. If you're behind a reverse proxy terminating TLS, don't set it; the proxy is presenting HTTPS to the browser and cookies work normally.

## The proper fix

Long-term, terminate TLS in front of the app. Options in rough order of "how much work":

1. **Cloudflare Tunnel or Tailscale Funnel** hand out publicly-trusted certificates automatically. Point the app at the tunnel URL and remove `INSECURE_COOKIES`.
2. **A real domain plus Let's Encrypt** via Caddy, Traefik, or nginx + certbot. Works even when the server is on an internal IP: use the DNS-01 challenge.
3. **A self-signed CA installed on every device that hits the app**. More setup, but no dependency on any outside service.

Full walkthrough at [HTTPS on the LAN](../self-hosting/lan-https.md).

Once TLS is in front of the app, remove `INSECURE_COOKIES` (or set it to `0`) and recreate the container. Cookies go back to `Secure: true`, browsers send them over the HTTPS connection, and everything works.

## Related

- [HTTPS on the LAN](../self-hosting/lan-https.md)
- [Reverse proxy](reverse-proxy.md)
- [Cloudflare Tunnel and Tailscale](../self-hosting/tunnels.md)
