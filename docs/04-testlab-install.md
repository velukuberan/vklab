# 04 - Testlab install

Goal: get the testlab code onto the server, create the two secret files, and start Traefik + Mailpit. After this doc, you can create WordPress sites.

## Set up SSH access to GitHub

The repo is private, so the server needs an SSH key registered with your GitHub account. Generate one on the server (not your laptop):

    ssh-keygen -t ed25519 -C "vklab-server" -f ~/.ssh/github_ed25519

Press Enter twice when it asks for a passphrase, or set one if you prefer the extra security.

Tell SSH to use this key for GitHub:

    cat >> ~/.ssh/config << 'EOF'
    Host github.com
      HostName github.com
      User git
      IdentityFile ~/.ssh/github_ed25519
      IdentitiesOnly yes
    EOF
    chmod 600 ~/.ssh/config

Show the public key:

    cat ~/.ssh/github_ed25519.pub

Copy the output, then go to https://github.com/settings/ssh/new and add it as a new authentication key. Title it something like "vklab server" so you can revoke it specifically later.

Test the connection:

    ssh -T git@github.com

You should see a "Hi <username>!" message confirming auth works.

## Clone the repo to /opt/testlab

The convention is to put system-level services under `/opt`. Clone directly there so the running system and the git repo are the same directory - edit once, changes reflect everywhere:

    sudo mkdir /opt/testlab
    sudo chown $USER:$USER /opt/testlab
    git clone git@github.com:<your-username>/vklab.git /opt/testlab
    cd /opt/testlab

## Create the Docker network

Traefik and every site share one Docker network called `web`. Traefik watches this network to auto-discover new containers:

    docker network create web

## Symlink the site management scripts

The `newsite`, `killsite`, and `siteinfo` scripts live in `bin/`. Symlink them into your PATH so you can run them from anywhere:

    sudo ln -s /opt/testlab/bin/newsite  /usr/local/bin/newsite
    sudo ln -s /opt/testlab/bin/killsite /usr/local/bin/killsite
    sudo ln -s /opt/testlab/bin/siteinfo /usr/local/bin/siteinfo

Verify:

    which newsite

Should print `/usr/local/bin/newsite`.

## Create the runtime state directory

Sites get created inside `sites/`. This directory is excluded from git because it's user data, not infrastructure, but it must exist:

    mkdir /opt/testlab/sites

## Create the Traefik secrets

The two example files in `traefik/` show what shape the real config needs to take.

**`traefik/.env`** - the DigitalOcean API token Traefik uses to solve DNS-01 challenges:

    cp traefik/.env.example traefik/.env
    chmod 600 traefik/.env
    vi traefik/.env

Replace `your_digitalocean_api_token_here` with the actual token from https://cloud.digitalocean.com/account/api/tokens. Save and exit.

**`traefik/dynamic.yml`** - basic auth for the Traefik dashboard:

Generate a bcrypt hash for the dashboard password (install `apache2-utils` first if needed):

    sudo apt install apache2-utils
    htpasswd -nbB admin 'your-strong-password'

Copy the output (looks like `admin:$2y$05$abc...`). Then:

    cp traefik/dynamic.yml.example traefik/dynamic.yml
    vi traefik/dynamic.yml

Replace the `admin:REPLACE_WITH_BCRYPT_HASH` line with the full line you copied. Save and exit.

Note: in this file, a single `$` is correct. Only docker-compose labels need `$$` to escape.

## Start Traefik

    cd /opt/testlab/traefik
    docker compose up -d

Watch the logs for the first minute to confirm Let's Encrypt certificates are being fetched:

    docker compose logs -f traefik

You want to see entries about certificate resolution succeeding. First cert takes 60-120 seconds because DigitalOcean DNS propagation is slow. Press Ctrl-C to stop tailing.

Then verify the dashboard is up at `https://traefik.test.yourdomain.com` (browser will prompt for the admin password you set).

## Start Mailpit

    cd /opt/testlab/mailpit
    docker compose up -d

Verify at `https://mail.test.yourdomain.com` - you should see an empty inbox.

## Test with a real site

    newsite wp1

Wait about 30 seconds, then visit `https://wp1.test.yourdomain.com`. You should land on the WordPress installer.

## Next

Testlab is running. Learn the daily commands - move on to [daily usage](05-daily-usage.md).
