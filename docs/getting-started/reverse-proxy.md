# Reverse proxy

Front the app with Caddy, nginx, or Traefik and let the proxy handle TLS, virtual hosts, and any subpath rewriting. Every Trace app behaves the same way behind a proxy; the only per-app difference is the internal port number.

!!! tip "Signed in, then sent back to the sign-in page?"
    See [If signing in doesn't stick](#sign-in-doesnt-stick) at the end of this page.

Internal ports as a reminder: each app listens on the same port inside the container as its sample host port. NutriTrace `3001`, LiftTrace `3002`, CookTrace `3003`, NoteTrace `3004`. (Before 1.3.0, LiftTrace listened on `3003` and CookTrace on `3001` inside the container; see [Updating](updating.md#lifttrace-and-cooktrace-130-container-port-change).)

## Root path (one app per hostname)

The simplest setup: each app gets its own subdomain (`cook.example.com`, `lift.example.com`, `nutri.example.com`) and lives at the root of that hostname. No env vars to change, no path rewriting.

=== "Caddy"

    ```caddy
    cook.example.com {
        reverse_proxy localhost:3003
    }

    lift.example.com {
        reverse_proxy localhost:3002
    }

    nutri.example.com {
        reverse_proxy localhost:3001
    }
    ```

    Caddy auto-issues Let's Encrypt certs as long as the hostnames resolve to the box.

=== "nginx"

    ```nginx
    server {
      listen 443 ssl http2;
      server_name cook.example.com;

      # ssl_certificate ...;
      # ssl_certificate_key ...;

      location / {
        proxy_pass http://localhost:3003;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
      }
    }
    ```

=== "Traefik"

    ```yaml
    services:
      cooktrace:
        image: ghcr.io/traceapps/cooktrace:latest
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.cooktrace.rule=Host(`cook.example.com`)"
          - "traefik.http.routers.cooktrace.entrypoints=websecure"
          - "traefik.http.routers.cooktrace.tls.certresolver=le"
          - "traefik.http.services.cooktrace.loadbalancer.server.port=3003"
    ```

## Subpath (multiple apps on one hostname)

If you'd rather run every app under one hostname (`example.com/cook/`, `example.com/lift/`, `example.com/nutri/`), set the `BASE_URL` env var on each container and configure the proxy to pass the prefix through **without stripping it**.

Set `BASE_URL` to the prefix, starting with `/` and with no trailing slash:

```yaml
environment:
  BASE_URL: /cooktrace
```

With `BASE_URL` set, the app's assets, API routes, service worker, and image URLs all live under that prefix. The proxy's job is to forward `/cooktrace/*` unchanged.

=== "Caddy"

    ```caddy
    example.com {
        handle /cooktrace/* {
            reverse_proxy localhost:3003
        }
        handle /lifttrace/* {
            reverse_proxy localhost:3002
        }
        handle /nutritrace/* {
            reverse_proxy localhost:3001
        }
    }
    ```

    Use `handle`, not `handle_path`. `handle_path` strips the matched prefix before proxying, which breaks every asset URL the app just emitted.

=== "nginx"

    ```nginx
    location /cooktrace/ {
      proxy_pass http://localhost:3003/cooktrace/;   # trailing slash on both sides
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
    }
    ```

    Trailing slashes on **both** sides of `proxy_pass` preserve the prefix. Drop either one and paths break.

=== "Traefik"

    ```yaml
    labels:
      - "traefik.http.routers.cooktrace.rule=Host(`example.com`) && PathPrefix(`/cooktrace`)"
      # Do NOT add a StripPrefix middleware. The app expects the prefix on every request.
    ```

    Traefik defaults are fine here; the trap is adding a `StripPrefix` middleware out of habit. Don't.

## The `BASE_URL` env var

`BASE_URL` is the one thing the app itself needs to know about the subpath, and it's the same var in every app. Rules:

- Must start with `/`. `BASE_URL=/cooktrace` is right; `BASE_URL=cooktrace` is not.
- Must not have a trailing slash. `BASE_URL=/cooktrace` is right; `BASE_URL=/cooktrace/` is not.
- Leave it unset (the default) to keep the app at root. This is the "one subdomain per app" case.
- Once set, every asset URL, API path, and service worker registration includes the prefix. Change it later and PWA clients need to re-register the service worker.

## WebSocket / upgrade headers

The sync channel, live progress bars during backup uploads, and long-running AI streams all rely on standard HTTP long-polling and `Upgrade` semantics. If your proxy strips `Upgrade`/`Connection` headers, sync stalls silently.

- Caddy passes upgrade headers by default; nothing to configure.
- nginx needs `proxy_set_header Upgrade $http_upgrade;` and `proxy_set_header Connection "upgrade";` (both included in the snippets above).
- Traefik passes upgrade headers by default.

## OIDC callbacks behind a proxy

When registering an OIDC provider with your IdP, the callback URL must be the **externally visible** one, including any subpath prefix:

```text
https://example.com/cooktrace/api/auth/oidc/callback/1
```

Enter the same URL in the app: in `OIDC_REDIRECT_URIS`, or in the provider's Redirect URIs under **Settings → Authentication**. The provider redirects the user back through the proxy to the prefixed path, and the app finishes the flow.

## If signing in doesn't stick { #sign-in-doesnt-stick }

Your password is accepted, then you're back on the sign-in page. NutriTrace 1.4.0 and NoteTrace 1.1.0 and later explain it on the sign-in screen: **Signing in won't work on this address** when the page is plain HTTP, or **Your browser didn't keep you signed in** when it's HTTPS (CookTrace and LiftTrace from their next release). Either way the browser threw away the sign-in cookie (`ct_token`, `lt_token`, `note_token` or `nt_token`). Check, in this order:

1. **The address bar says `https://`.** At a plain `http://` address, the proxy's own HTTP port included, the browser drops the cookie because it is HTTPS-only. Open the `https://` address, or have the proxy redirect HTTP to HTTPS (Caddy does on its own). On a LAN without TLS, see [Plain-HTTP LAN](lan-http.md).
2. **Cookies are allowed for the site.** Blocked cookies, a privacy extension or strict private-window settings drop it.
3. **The app is open in its own tab.** Inside a frame on another site (a dashboard at another address), the browser can refuse the cookie.
4. **The proxy passes `Set-Cookie` through unchanged.** No rule should strip it or rewrite its domain. With a subpath, forward the prefix unchanged, as above.

To see where it goes wrong, sign in with the browser's developer tools open on the Network tab. The sign-in response should set the cookie named above, with `HttpOnly`, `Secure` and `SameSite=Lax`, and the next request should send it back. Firefox's console names a cookie it rejected.

## Public links behind a login { #public-links-behind-a-login }

If your proxy asks for its own login in front of the app (basic auth, Authelia, an Authentik proxy outpost, Cloudflare Access), it stops everyone who opens a public link too: a NoteTrace note link or a CookTrace recipe link. To let people without that login open them, leave these paths open and keep everything else behind the login:

| App | Leave open |
|---|---|
| NoteTrace (1.1.0 and later) | `/n/`, `/api/n/` |
| CookTrace (from its next release) | `/r/`, `/api/r/` |
| Both | `/assets/`, `/fonts/`, `/icons/`, `/uploads/` |

The first two rows are the link page and the note or recipe it shows. The last row is the page's own scripts, fonts and icons, and the pictures on the note or recipe. These paths only read. Leaving `/uploads/` open makes every uploaded file reachable by its address, not only the pictures on shared notes: the app names them so they can't be guessed, but anyone who has an address can open it without the proxy's login. `/manifest.webmanifest` may be left open too; without it a stranger's browser only logs an error. Earlier CookTrace links look like `/#/r/...`, which a proxy can't tell apart from the rest of the app, so they need the app's page open as well.

You open your own links with the proxy's login as usual (NoteTrace and CookTrace from their next release; until then, NoteTrace's page says **Couldn't Load the Note** behind such a proxy).

=== "nginx"

    ```nginx
    # Public links: no login.
    location ~ ^/(n|api/n|assets|fonts|icons|uploads)/ {
        proxy_pass http://notetrace:3004;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    # Everything else: the proxy's login, plus the usual settings from above.
    location / {
        auth_basic "NoteTrace";
        auth_basic_user_file /etc/nginx/htpasswd;
        proxy_pass http://notetrace:3004;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    ```

    Behind a subpath (`BASE_URL=/notetrace`), the open location becomes `location ~ ^/notetrace/(n|api/n|assets|fonts|icons|uploads)/` with `proxy_pass http://notetrace:3004;` and no path after the port: nginx refuses one in a regex location, and the app expects the subpath kept.

=== "Authelia"

    Put the rule above every other rule that matches the host, wildcards like `*.example.com` included: the first rule that matches wins.

    ```yaml
    access_control:
      rules:
        - domain: 'notes.example.com'
          policy: 'bypass'
          resources:
            - '^/(n|api/n|assets|fonts|icons|uploads)/.*$'
    ```

For CookTrace, use `r|api/r` in place of `n|api/n`.

## Deeper recipes

Per-proxy walkthroughs, Cloudflare Tunnel setup, Tailscale Funnel, self-signed CA, and Android certificate trust all live in [Reverse-proxy recipes](../self-hosting/proxy-recipes.md). This page covers the essentials that apply everywhere; that page covers the specifics.

## Related

- [Reverse-proxy recipes](../self-hosting/proxy-recipes.md)
- [HTTPS on the LAN](../self-hosting/lan-https.md)
- [Cloudflare Tunnel and Tailscale](../self-hosting/tunnels.md)
- [OIDC / SSO overview](../auth/oidc.md)
