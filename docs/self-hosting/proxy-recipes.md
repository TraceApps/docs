# Reverse-proxy recipes

Drop-in configs for the three proxies people actually use. Each block assumes the app container is reachable at `cooktrace:3001` on a shared Docker network, with a public hostname of `cook.example.com`. Swap the service name and port for LiftTrace (`lifttrace:3003`) or NutriTrace (`nutritrace:3001`).

All three examples cover the same four things: TLS termination, correct `X-Forwarded-*` headers so the app knows its public URL, a raised upload-size cap for full-backup restores, and websocket upgrade headers (used by the settings-sync live channel and, in LiftTrace, the radio player metadata stream).

## Root-domain install

=== "Caddy"

    ```caddy
    cook.example.com {
        encode gzip zstd

        # Raise the request body cap for backup restores; default is 10 MiB.
        request_body {
            max_size 512MB
        }

        reverse_proxy cooktrace:3001 {
            header_up X-Forwarded-Proto {scheme}
            header_up X-Forwarded-Host  {host}
            header_up X-Real-IP         {remote_host}
        }
    }
    ```

    Caddy handles ACME certs automatically. Websocket upgrade is on by default; no extra directive needed. Use `handle` (not `handle_path`) if you later add a subpath block, since the app expects the full request URI including `BASE_URL`.

=== "nginx"

    ```nginx
    server {
        listen 443 ssl http2;
        server_name cook.example.com;

        ssl_certificate     /etc/letsencrypt/live/cook.example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/cook.example.com/privkey.pem;

        # 512 MB matches BACKUP_UPLOAD_MAX_MB; raise both together if you go higher.
        client_max_body_size 512M;

        location / {
            proxy_pass http://cooktrace:3001;

            proxy_set_header Host              $host;
            proxy_set_header X-Real-IP         $remote_addr;
            proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host  $host;

            # Websocket upgrade for live sync / radio metadata.
            proxy_http_version 1.1;
            proxy_set_header Upgrade    $http_upgrade;
            proxy_set_header Connection "upgrade";

            # Backups can take a minute or two to stream on slow links.
            proxy_read_timeout 300s;
            proxy_send_timeout 300s;
        }
    }
    ```

    Also open `listen 80` with a redirect to 443, or let certbot's snippet handle it.

=== "Traefik"

    Labels on the app container, no extra config file needed:

    ```yaml
    services:
      cooktrace:
        image: ghcr.io/traceapps/cooktrace:1
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.cooktrace.rule=Host(`cook.example.com`)"
          - "traefik.http.routers.cooktrace.entrypoints=websecure"
          - "traefik.http.routers.cooktrace.tls.certresolver=letsencrypt"
          - "traefik.http.services.cooktrace.loadbalancer.server.port=3001"
          # Raise the buffering cap on the entrypoint (see Traefik static config)
          # or add a per-router middleware; Traefik's default is 512 MB body but
          # only 4 KB response headers, so no per-router tweak is usually needed.
        networks:
          - proxy
    ```

    Traefik streams websockets automatically. Do not add `stripprefix` middleware unless you also set `BASE_URL=/cooktrace` inside the container; the app expects the URI it was told about.

## Subpath install (`BASE_URL`)

Mount the app at `/cooktrace` on an existing domain by setting `BASE_URL=/cooktrace` in the container environment and matching that in the proxy. The app emits every asset URL and every API call under that prefix; the proxy must forward the prefix through intact.

=== "Caddy"

    ```caddy
    tools.example.com {
        handle /cooktrace/* {
            reverse_proxy cooktrace:3001 {
                header_up X-Forwarded-Proto {scheme}
                header_up X-Forwarded-Host  {host}
            }
        }
        handle {
            # your other services / static site here
        }
    }
    ```

    `handle` (not `handle_path`) preserves the `/cooktrace` prefix in the upstream request, which is what the app wants.

=== "nginx"

    ```nginx
    location /cooktrace/ {
        proxy_pass http://cooktrace:3001;   # trailing slash preserves prefix
        proxy_set_header Host              $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host  $host;
        client_max_body_size 512M;
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    ```

    Keep the trailing slash on both `location` and `proxy_pass`, and set `BASE_URL=/cooktrace` in the container. Do not add a `rewrite` that strips the prefix.

=== "Traefik"

    ```yaml
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.cooktrace.rule=Host(`tools.example.com`) && PathPrefix(`/cooktrace`)"
      - "traefik.http.routers.cooktrace.entrypoints=websecure"
      - "traefik.http.routers.cooktrace.tls.certresolver=letsencrypt"
      - "traefik.http.services.cooktrace.loadbalancer.server.port=3001"
      # NOTE: no stripprefix middleware. The app expects to see /cooktrace.
    ```

    Set `BASE_URL=/cooktrace` in the container `environment:` block alongside the labels.

## Verifying the proxy is doing its job

After reload, curl the app through the proxy and check two things: the scheme in the response body reflects `https`, and cookies come back with the `Secure` attribute.

```bash
curl -sSI https://cook.example.com/ | grep -i 'set-cookie\|content-type'
```

If cookies come back without `Secure`, `X-Forwarded-Proto` is not reaching the app. If login appears to succeed but the next request 401s, the cookie is being set but not sent, usually because `INSECURE_COOKIES` is unset while the browser is talking plain HTTP.

## Related

- [Environment reference](env-vars.md)
- [HTTPS on the LAN](lan-https.md)
- [Cloudflare Tunnel and Tailscale](tunnels.md)
