# 05 - Daily usage

Three commands run the whole show: `newsite`, `killsite`, `siteinfo`.

## Create a site

    newsite wordpress wp1

Creates a WordPress site at `https://wp1.test.yourdomain.com` with the latest WordPress and PHP 8.3. Ready in about 10 seconds.

You can pin versions:

    newsite wordpress wp2 --wp 6.4 --php 8.1

Available flags:

- `--wp <version>` - WordPress version (default: latest)
- `--php <version>` - PHP version, must be one supported by the WordPress image

The script auto-installs WordPress via WP-CLI, so the site is fully configured on first load - no manual installer walkthrough.

## Destroy a site

    killsite wp1

Stops the containers, removes the volumes, deletes the site directory, and removes the site from Traefik's routing table.

Warning: this is not reversible. All site data (DB, uploads, plugins) is deleted.

## Get info about a site

    siteinfo wp1

Prints the URL, admin username, admin password, admin email, and container status. Handy when you forgot the credentials for a site you set up yesterday.

Flags:

- `--password-only` - print just the password. Useful for piping to a clipboard tool.

For example, if you have a `cpy()` bash function that pipes to your clipboard (see [troubleshooting](07-troubleshooting.md) for setup):

    siteinfo wp1 --password-only | cpy

Then Ctrl-V pastes the password into the browser login.

## Check what's running

    docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'

Shows all containers - Traefik, Mailpit, socket-proxy, and every site.

## Check what sites exist

    ls /opt/testlab/sites/

Each subdirectory is one site.

Or check `/opt/testlab/sites.log` for a chronological record of every `newsite` and `killsite` event.

## Check the Traefik dashboard

Visit `https://traefik.test.yourdomain.com`. It shows every route Traefik knows about, live. If a site isn't reachable, this is the first place to look - if the router isn't listed, Traefik doesn't know about it, which usually means the container isn't on the `web` network or its labels are wrong.

## Check caught emails

Visit `https://mail.test.yourdomain.com`. Every email sent from every WordPress site lands here. The From address tells you which site sent it.

## Cost management

The server bills whether or not you're actively using it, including when powered off. To minimize cost during long breaks:

1. Take a snapshot of the server (via your cloud provider's UI)
2. Destroy the server
3. When you want to work again, create a new server from the snapshot
4. Update the DNS wildcard A record to the new IP
5. Everything comes back up automatically thanks to Docker's `restart: unless-stopped` policy

Snapshots typically cost around $0.06/GB/month - much less than an idle server.

## Next

If something breaks, check [troubleshooting](07-troubleshooting.md). If you lose the server entirely and want to rebuild from nothing, see [disaster recovery](06-disaster-recovery.md).
