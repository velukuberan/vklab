# 06 - Disaster recovery

## First step on every restore: update DNS

**Do this before anything else.** After creating the droplet from snapshot, DigitalOcean assigns a new public IPv4. Every subdomain (`traefik`, `mail`, and every `wpN` site) will be unreachable until DNS points to the new IP.

### Steps

1. In DigitalOcean → **Droplets** → your restored droplet → copy the **Public IPv4** (top-right).

2. In DigitalOcean → **Networking → Domains → test.vkuberan.in**.

3. Update every A record to the new IP:
   - `*` (wildcard) → new IP
   - `@` (apex, if present) → new IP
   - Any individually-named records (e.g. `traefik`, `mail`) — usually not needed if `*` covers them, but check.

4. Verify from your laptop:

        dig +short traefik.test.vkuberan.in
        # should return the new IP within ~60 seconds

5. Only after `dig` returns the new IP → proceed to SSH into the droplet and start work.

### Why this matters

- Skipping this step means "nothing works" on the restored droplet — Traefik dashboard, mailpit, every WP site — because DNS points to the old dead IP.
- Diagnosing without knowing about this can burn 30+ minutes chasing symptoms that look like Traefik / cert / networking bugs.
- Docker containers themselves come up fine; the problem is entirely on the DNS side.

### One future upgrade

A DigitalOcean **Floating IP** eliminates this step entirely — attach it to whichever droplet is current, DNS never changes. It's free while attached to a droplet ($4/mo only during unattached periods). Consider it if destroy-restore becomes a weekly habit.

"Disaster" here means: the server is gone, the snapshot is gone, and all you have is this git repo. The goal is a working testlab from nothing in about 30-45 minutes.

Sites you had running are not recoverable this way - `sites/` is excluded from git because it's user data, not infrastructure. If you want that too, keep a separate backup of `sites/` somewhere (S3, another server, a snapshot of just that directory).

## The recovery path

Follow the standard install docs in order:

1. [Prerequisites](01-prerequisites.md) - if you kept the same domain and cloud account, this is already done. Just make sure your API token is still valid, or generate a new one.
2. [Server setup](02-server-setup.md) - fresh Ubuntu server, non-root user, Docker, UFW, swap.
3. [DNS setup](03-dns-setup.md) - if the subdomain is still delegated to your cloud provider, just update the wildcard A record to the new server's IP. If not, redo delegation.
4. [Testlab install](04-testlab-install.md) - clone repo, create secrets, start Traefik and Mailpit.

## Things that make this faster

**A password manager entry for the testlab.** Store the DO API token, Traefik dashboard password, and any recurring config in one place. Recovery becomes copy-paste from the vault, not hunting through emails.

**A saved SSH config on your laptop.** An entry like this in `~/.ssh/config` means you don't need to remember the IP:

    Host vklab
      HostName <server-ip>
      User vkuberan
      IdentityFile ~/.ssh/id_ed25519

Then `ssh vklab` just works. When you change the IP after recovery, update this one file.

**Notes in the repo.** If you customize anything unusual on the server (kernel tunings, extra tools, cron jobs), add them to a doc here. Future-you will thank you.

## What can go wrong during recovery

**DNS hasn't propagated yet.** After delegating a subdomain, it can take up to an hour for the new nameservers to be visible worldwide. If Traefik keeps failing to get certs, wait and retry - don't hammer Let's Encrypt or you'll hit rate limits.

**Let's Encrypt rate limits.** Production has a limit of 5 duplicate certificates per week. If you spent the previous evening iterating on Traefik config and burned through your quota, switch to LE staging temporarily (change the CA server URL in `traefik/traefik.yml`) - it has no meaningful rate limit but issues untrusted certs, so it's only for testing that the flow works.

**Firewall not open.** UFW must allow 80/tcp and 443/tcp before Traefik can bind them. Check with `sudo ufw status`.

**Docker permission denied.** If you get "permission denied while trying to connect to the Docker daemon socket" after installing Docker, you forgot to log out and back in after `usermod -aG docker $USER`.

## Restoring site data (optional)

If you kept a backup of `sites/` from before the disaster:

1. Complete the full install above so Traefik and Mailpit are running.
2. Copy the backed-up `sites/` directory into `/opt/testlab/sites/`.
3. For each site subdirectory, `cd` into it and run `docker compose up -d`.

The Traefik labels in each site's `docker-compose.yml` mean Traefik will auto-discover them and route traffic without additional config. Certificates for existing site hostnames get re-issued automatically.

### Restore git signing + agent (if starting from a clean droplet, not a snapshot)

Snapshot restores already have `keychain`, git config, and the SSH key baked in — nothing to do.

For a **from-scratch rebuild** (no snapshot available), re-run the "Git: passwordless pushes + signed commits" section from `04-testlab-install.md`. The private key itself must be re-generated (or restored from your offline backup) and re-registered on GitHub as both an Authentication Key and a Signing Key.
