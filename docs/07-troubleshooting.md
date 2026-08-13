# 07 - Troubleshooting

Things that broke during setup and how to fix them, plus a few personal-tooling notes.

## Traefik won't start

Check the logs:

    cd /opt/testlab/traefik
    docker compose logs -f traefik

**"Docker API version mismatch"** - Traefik versions below 3.5 hard-code an old Docker API path that modern Docker rejects. Solution: use Traefik v3.7 or later (already pinned in the compose file, but check if you ever updated it).

**"acme: error presenting token"** during cert challenges - Let's Encrypt can't verify the DNS TXT records because DigitalOcean DNS propagation is slow. Fix: set `propagation.delayBeforeChecks: 120s` in `traefik.yml`. Already set in the shipped config.

**Dashboard login rejected** - the bcrypt hash escaping is different between config files and docker-compose labels:

- In `traefik/dynamic.yml` (mounted config file): use single `$` - `admin:$2y$05$...`
- In `docker-compose.yml` labels: use `$$` - `admin:$$2y$$05$$...`

If you copied the hash between the two and forgot to adjust, login fails silently.

## Site creation fails

**"invalid image tag"** - `wordpress:latest-php8.3-apache` does not exist. The correct pattern is `wordpress:php8.3-apache` (for latest WP with pinned PHP). This is already fixed in `newsite`, but if you edit the script, watch out.

**"randstr: SIGPIPE"** - happens when bash `pipefail` combines with `/dev/urandom | head -c`. The `head` closes the pipe early, `urandom` gets SIGPIPE, and pipefail treats it as an error. Fix in the script: temporarily disable pipefail around that line, or use `tr -dc 'A-Za-z0-9' < /dev/urandom | head -c N` which handles it more gracefully. Already fixed in `newsite`.

## newsite hangs at "Waiting for..."

If newsite prints "Waiting for https://<site>.test.<domain> to respond..." and never proceeds, one of two things is happening:

**Site actually isn't up.** Check `docker ps | grep <site>-wp` — if the container isn't running, look at `docker logs <site>-wp`.

**Site is up but curl-from-droplet can't reach it.** This is hairpin NAT: Docker doesn't permit a container to reach the host's own public IP via loopback. The site works fine from a browser (external network) but hangs from the droplet itself.

Verify by testing both paths from the droplet:

    # This will hang if hairpin is the issue
    curl -kI https://wp1.test.yourdomain.com

    # This bypasses the loop — should return 200/302 instantly
    curl -kI --resolve wp1.test.yourdomain.com:443:127.0.0.1 https://wp1.test.yourdomain.com

If the second one works and the first doesn't → hairpin. `newsite` already handles this with `--resolve`, so if you're seeing this in the wait loop it means someone edited that line out. Restore it.

## SSL cert has the wrong name

If the browser shows a warning that the cert doesn't match the hostname, Traefik probably fell back to the default self-signed cert because the wildcard cert wasn't issued yet. Check `docker compose logs traefik | grep -i acme` for errors. Common causes:

- DO API token missing or wrong scope (needs read + write on DNS)
- Domain not delegated to DO yet (`dig NS test.yourdomain.com` should show DO nameservers)
- Rate limited by Let's Encrypt

## Powered-off server still billing

Powered-off servers still bill at full rate on most clouds, because your CPU/RAM/disk stays reserved for you. To actually stop billing:

1. Snapshot the server
2. Destroy the server
3. Restore from snapshot when you need it again

Snapshots cost around $0.06/GB/month, orders of magnitude less than an idle server.

## Personal tooling notes

These are not part of the testlab itself but I use them alongside it:

- **tmux** with TPM + tmux-resurrect + tmux-continuum - persistent sessions that survive SSH disconnects
- **fzf** - fuzzy file finder, mostly for jumping between site directories
- **tree** - visual directory listings
- **Alacritty** terminal on my laptop - supports OSC 52 clipboard escape sequences, which lets me copy text out of an SSH session with a shell function:

      cpy() {
        local input
        if [ -t 0 ]; then input="$*"; else input=$(cat); fi
        printf '\033]52;c;%s\033\\' "$(printf '%s' "$input" | base64 | tr -d '\n')"
      }

  Added to the server's `~/.bashrc`. Then `siteinfo wp1 --password-only | cpy` copies the password to my laptop's clipboard through the SSH connection. GNOME Terminal does not support OSC 52; Alacritty, iTerm2, WezTerm, and Kitty do.

None of these are required for the testlab to work. They just make daily use nicer.
