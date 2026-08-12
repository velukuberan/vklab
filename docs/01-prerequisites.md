# 01 - Prerequisites

Before you start, you'll need the following. Total setup cost: about $12/month plus a domain name if you don't already have one.

## Accounts and services

- **A cloud provider account** where you can create a virtual server with a public IPv4 address. This project was built on DigitalOcean, but the setup is cloud-agnostic - you can use Linode, Hetzner, AWS, or even a Raspberry Pi at home. Whatever you pick, keep 2 GB RAM and 50 GB disk as your minimum.
- **A domain you control.** Any registrar is fine (GoDaddy, Namecheap, Cloudflare, etc.). You do not need to move the whole domain to your cloud - we'll only delegate a subdomain like `test.yourdomain.com` in the DNS setup step.
- **An API token from your cloud provider's DNS service** with read + write scope. Traefik uses this to prove ownership of your subdomain when requesting Let's Encrypt certificates via the DNS-01 challenge. On DigitalOcean, generate one at https://cloud.digitalocean.com/account/api/tokens.

## Local tools

You'll be doing everything over SSH, so you need:

- An SSH client (built into macOS, Linux, and modern Windows via WSL or PowerShell).
- An SSH key pair. If you don't have one, generate it with `ssh-keygen -t ed25519 -C "your@email.com"`. Add the public key to your cloud provider's SSH keys section so new droplets accept your login.

## Skills assumed

You don't need to be an expert, but this walkthrough assumes you're comfortable:

- Running commands in a Linux shell.
- Editing files with `vi`, `nano`, or similar.
- Understanding what "restart a service" means at a basic level.

If any of the commands in later docs feel opaque, ask - most of them have a reason worth understanding, not just copying.

## Next

Once you have your cloud account, domain, and SSH key ready, move on to [server setup](02-server-setup.md).
