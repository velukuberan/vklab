# vklab

Self-hosted lab for spinning up disposable web app sandboxes with Traefik + Docker.

Currently supports WordPress. Django and FastAPI templates planned.

## What this does

One command creates a full stack behind HTTPS with a real cert:

    newsite wp1
    # -> https://wp1.test.vkuberan.in is live in ~30 seconds

One command tears it down:

    killsite wp1

Shared services (Traefik reverse proxy, Mailpit email catcher) route traffic and catch outbound mail from all sites.

## Architecture

                      Internet
                         |
                         v
            +------------------------+
            |   Traefik (80/443)     |  <- wildcard Let's Encrypt cert
            +-----------+------------+
                        |
         +--------------+--------------+--------------+
         |              |              |              |
         v              v              v              v
      +-----+        +-----+       +---------+    +-------+
      | wp1 |        | wp2 |       | mailpit |    |  ...  |
      +-----+        +-----+       +---------+    +-------+
                       Docker network: web

## Status

Work in progress. README will be expanded as the setup stabilizes.

See individual directories for now:

- traefik/    - reverse proxy + wildcard HTTPS
- mailpit/    - shared email catcher
- templates/wordpress/ - per-site compose template
- bin/        - newsite, killsite, siteinfo scripts
