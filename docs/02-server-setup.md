# 02 - Server setup

Goal: go from a fresh Ubuntu 24 server to one that has Docker installed, a firewall enabled, a non-root user, and enough swap to handle memory pressure.

## Create the server

Provision an Ubuntu 24 LTS server with at least 2 GB RAM and 50 GB disk. On DigitalOcean this is the $12/month "Basic" plan. Add your SSH key during creation so you can log in without a password.

Once it's up, SSH in as root:

    ssh root@<your-server-ip>

## Create a non-root user

Running everything as root is a bad habit. Create a personal user with sudo access:

    adduser vkuberan
    usermod -aG sudo vkuberan

Copy the SSH key from root so you can log in as the new user:

    rsync --archive --chown=vkuberan:vkuberan ~/.ssh /home/vkuberan

Log out, then log back in as the new user to confirm it works:

    ssh vkuberan@<your-server-ip>

From here on, everything runs as this user. Root SSH login can optionally be disabled later.

## Add swap

A 2 GB server can run out of memory when Docker pulls images or WordPress does something greedy. Adding 2 GB of swap avoids sudden crashes:

    sudo fallocate -l 2G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

The last line makes swap persist across reboots.

## Enable the firewall

UFW (Uncomplicated Firewall) blocks everything by default and lets in only what you allow:

    sudo ufw allow OpenSSH
    sudo ufw allow 80/tcp
    sudo ufw allow 443/tcp
    sudo ufw enable

Verify with `sudo ufw status` - you should see OpenSSH, 80/tcp, and 443/tcp all allowed from anywhere.

## Install Docker

Follow the official Docker install for Ubuntu at https://docs.docker.com/engine/install/ubuntu/. The steps change over time, so linking rather than copying is safer.

After installation, add your user to the `docker` group so you don't need sudo for every Docker command:

    sudo usermod -aG docker $USER

You need to log out and back in for the group change to take effect. Confirm it worked:

    docker ps

If you see an empty table with headers instead of a permission error, you're good.

## Next

Server ready. Now delegate DNS so Let's Encrypt can prove you own your subdomain - move on to [DNS setup](03-dns-setup.md).
