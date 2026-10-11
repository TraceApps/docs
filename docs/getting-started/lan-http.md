# Plain-HTTP LAN (INSECURE_COOKIES)

This is the first-time gotcha almost every self-hoster hits. You open the app at a plain `http://` address, such as `http://192.168.1.20:3001` (or `:3002` / `:3003` / `:3004`) or an Unraid WebUI link, sign in, and end up right back on the sign-in page. The password was right.

The cause is cookies, not credentials.

## What you'll see

- **NutriTrace 1.4.0 and NoteTrace 1.1.0 and later** say so before you even try. The sign-in and setup screens show **Signing in won't work on this address**, with a **How to fix it** link to this page, and the server log says it too, at most once an hour:

    ```text
    [WARN] Sign-in from a plain-HTTP page (http://192.168.1.20:3001). The sign-in cookie only works over HTTPS, so the browser drops it and the user lands back on the login page. Serve NutriTrace over HTTPS, or set INSECURE_COOKIES=1 on a trusted LAN. See https://traceapps.github.io/docs/getting-started/lan-http/
    ```

- **CookTrace and LiftTrace** do the same from their next release. Until then they send you back to the sign-in page with no explanation (LiftTrace can open the app first, with nothing in it, until you reload), and nothing in the server log says why.

`http://localhost`, opened on the server itself, is the exception in Chrome and Firefox: they treat it as secure and keep the cookie, so signing in there works without any of this. Safari may not.

## Why cookies fail over plain HTTP

By default the app issues its sign-in cookie with the `Secure` attribute set, which tells the browser: "only keep this cookie on HTTPS." Reach the app over plain `http://` and the browser silently throws the cookie away. The next request has no session, so the app sends you back to the sign-in page. From the outside it looks like a bad password. Firefox's console names the cookie it rejected: `Cookie “nt_token” has been rejected because a non-HTTPS cookie can’t be set as “secure”.`

This is the browser doing exactly what it's supposed to. The default is `Secure` because that's the safe choice for anything reachable from outside a trusted network.

The Android app signs in with a token, not this cookie, so `INSECURE_COOKIES` makes no difference to it. For the app over plain HTTP, see [HTTPS on the LAN](../self-hosting/lan-https.md#path-4-sideload-the-debug-apk).

## The escape hatch: `INSECURE_COOKIES=1`

Set the `INSECURE_COOKIES` env var to `1` and the server issues the cookie without the `Secure` attribute. The browser then keeps it over plain HTTP and signing in works.

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

=== "NoteTrace"

    ```yaml
    services:
      notetrace:
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

Then `docker compose up -d` to recreate the container. `INSECURE_COOKIES=1` in `.env` works too when your compose file passes it to the container: with `env_file: .env`, or `INSECURE_COOKIES=${INSECURE_COOKIES:-}` under `environment:`. The sample `docker-compose.yml` in each app's repo does.

!!! danger "Only for trusted LAN, never for internet-exposed installs"
    `INSECURE_COOKIES=1` strips the single browser-side defense that keeps your session token off open WiFi. Use it only when the app is reachable exclusively from a network you control (your home LAN, a Tailscale mesh, a VPN). Never on anything that resolves from the public internet, even briefly. If you're behind a reverse proxy terminating TLS, leave it at `0`: the proxy presents HTTPS to the browser and the cookie works normally.

## The proper fix

Long-term, terminate TLS in front of the app. Options in rough order of "how much work":

1. **Cloudflare Tunnel or Tailscale Funnel** hand out publicly-trusted certificates automatically. Point the app at the tunnel URL and remove `INSECURE_COOKIES`.
2. **A real domain plus Let's Encrypt** via Caddy, Traefik, or nginx + certbot. Works even when the server is on an internal IP: use the DNS-01 challenge.
3. **A self-signed CA installed on every device that hits the app**. More setup, but no dependency on any outside service.

Full walkthrough at [HTTPS on the LAN](../self-hosting/lan-https.md).

Once TLS is in front of the app, remove `INSECURE_COOKIES` (or set it to `0`) and recreate the container. The cookie goes back to `Secure`, browsers keep it over the HTTPS connection, and everything works. Still sent back to the sign-in page over HTTPS? See [If signing in doesn't stick](reverse-proxy.md#sign-in-doesnt-stick).

## Related

- [HTTPS on the LAN](../self-hosting/lan-https.md)
- [Reverse proxy](reverse-proxy.md)
- [Cloudflare Tunnel and Tailscale](../self-hosting/tunnels.md)
