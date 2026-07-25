# Reverse proxy

Front the app with Caddy, nginx, or Traefik and let the proxy handle TLS, virtual hosts, and any subpath rewriting. All three apps behave the same way behind a proxy; the only per-app difference is the internal port number.

Internal ports as a reminder: CookTrace and NutriTrace listen on `3001`, LiftTrace on `3003`.

## Root path (one app per hostname)

The simplest setup: each app gets its own subdomain (`cook.example.com`, `lift.example.com`, `nutri.example.com`) and lives at the root of that hostname. No env vars to change, no path rewriting.

=== "Caddy"

    ```caddy
    cook.example.com {
        reverse_proxy localhost:3000
    }

    lift.example.com {
        reverse_proxy localhost:3002
    }

    nutri.example.com {
        reverse_proxy localhost:3000
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
        proxy_pass http://localhost:3000;
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
        image: ghcr.io/traceapps/cooktrace:1
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.cooktrace.rule=Host(`cook.example.com`)"
          - "traefik.http.routers.cooktrace.entrypoints=websecure"
          - "traefik.http.routers.cooktrace.tls.certresolver=le"
          - "traefik.http.services.cooktrace.loadbalancer.server.port=3001"
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
            reverse_proxy localhost:3000
        }
        handle /lifttrace/* {
            reverse_proxy localhost:3002
        }
        handle /nutritrace/* {
            reverse_proxy localhost:3000
        }
    }
    ```

    Use `handle`, not `handle_path`. `handle_path` strips the matched prefix before proxying, which breaks every asset URL the app just emitted.

=== "nginx"

    ```nginx
    location /cooktrace/ {
      proxy_pass http://localhost:3000/cooktrace/;   # trailing slash on both sides
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

`BASE_URL` is the one thing the app itself needs to know about the subpath, and it's the same var across all three apps. Rules:

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

Enter the same URL in **Settings** then **User Management** then **OIDC providers**. The provider redirects the user back through the proxy to the prefixed path, and the app finishes the flow.

## Deeper recipes

Per-proxy walkthroughs, Cloudflare Tunnel setup, Tailscale Funnel, self-signed CA, and Android certificate trust all live in [Reverse-proxy recipes](../self-hosting/proxy-recipes.md). This page covers the essentials that apply everywhere; that page covers the specifics.

## Related

- [Reverse-proxy recipes](../self-hosting/proxy-recipes.md)
- [HTTPS on the LAN](../self-hosting/lan-https.md)
- [Cloudflare Tunnel and Tailscale](../self-hosting/tunnels.md)
- [OIDC / SSO overview](../auth/oidc.md)
