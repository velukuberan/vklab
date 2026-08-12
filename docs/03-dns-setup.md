# 03 - DNS setup

Goal: point a wildcard subdomain like `*.test.yourdomain.com` to your server, and delegate DNS management for that subdomain to your cloud provider so Traefik can request certs automatically.

## The idea

Traefik uses the DNS-01 challenge to get wildcard SSL certificates from Let's Encrypt. This means it needs to create temporary TXT records on your DNS to prove ownership. Since it uses your cloud provider's API to do this, your DNS must be hosted at that cloud provider.

You don't have to move your whole domain - just delegate a subdomain. Your main site stays wherever it is.

## Delegate the subdomain

Pick a subdomain to use for the testlab, for example `test.yourdomain.com`. Then:

**On your cloud provider** (DigitalOcean example, but the concept is the same on any provider):

1. Go to https://cloud.digitalocean.com/networking/domains
2. Add the subdomain `test.yourdomain.com`
3. DO shows you three nameservers: `ns1.digitalocean.com`, `ns2.digitalocean.com`, `ns3.digitalocean.com`. Keep this tab open.

**At your registrar** (GoDaddy, Namecheap, etc.):

1. Open DNS management for your domain
2. Add three NS records for the subdomain:
   - Host: `test`, Value: `ns1.digitalocean.com`
   - Host: `test`, Value: `ns2.digitalocean.com`
   - Host: `test`, Value: `ns3.digitalocean.com`
3. Save

DNS delegation typically propagates within an hour but can take up to 24 hours. You can check with `dig NS test.yourdomain.com` - you should see the DO nameservers listed.

## Add the wildcard A record

Back on your cloud provider's DNS panel for the subdomain:

1. Add an A record with:
   - Hostname: `*`
   - IP: your server's public IPv4
   - TTL: 3600 (or default)
2. Add another A record with hostname `@` (the apex) pointing to the same IP - optional, but nice to have.

The wildcard `*` means anything you spin up - `wp1.test.yourdomain.com`, `mail.test.yourdomain.com`, `django3.test.yourdomain.com` - all point to your server automatically. No DNS work when creating new sites.

## Verify

    dig +short wp1.test.yourdomain.com

Should return your server's IP. If not, wait a few more minutes and retry.

## Next

DNS is pointing at your server. Now install and configure the testlab itself - move on to [testlab install](04-testlab-install.md).
