# vklab

Self-hosted lab for spinning up disposable web app sandboxes with Traefik + Docker.

Currently supports WordPress. Django and FastAPI templates planned.

## What this does

One command creates a full stack behind HTTPS with a real cert:

    newsite wp1
    # -> https://wp1.test.vkuberan.in is live in ~10 seconds

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

## Getting Started

Follow the docs in order for a fresh install or full restore:

1. [Prerequisites](docs/01-prerequisites.md) - what you need before starting
2. [Server setup](docs/02-server-setup.md) - Ubuntu, Docker, UFW, swap, user
3. [DNS setup](docs/03-dns-setup.md) - delegating a subdomain to your cloud DNS
4. [Testlab install](docs/04-testlab-install.md) - clone repo, secrets, start services
5. [Daily usage](docs/05-daily-usage.md) - creating and destroying sites
6. [Disaster recovery](docs/06-disaster-recovery.md) - restoring from a total loss
7. [Troubleshooting](docs/07-troubleshooting.md) - things that broke and how to fix them

## Status

Work in progress. Currently running on DigitalOcean but designed to be cloud-agnostic - see individual docs for the shallow assumptions.

Repo layout:

- traefik/    - reverse proxy + wildcard HTTPS config
- mailpit/    - shared email catcher
- templates/wordpress/ - per-site compose template
- bin/        - newsite, killsite, siteinfo scripts
- docs/       - setup and usage documentation
