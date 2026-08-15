# Droplet disk usage — findings (2026-08-15)

## Symptom
DO snapshot grew from ~8GB to ~16GB.

## Root cause
Leftover Docker images from past "wp turn up" test cycles were never pruned.
`docker system df -v` showed 19 images on disk but only 3 containers running
(mailpit, traefik, socket-proxy) — no WordPress or MariaDB container was
running at investigation time, yet 13 different wordpress:* image tags and
mariadb:11 were still on disk with 0 containers attached.

This droplet uses Docker's containerd image store (`Storage Driver: overlayfs`
in `docker info`, not the older `overlay2`). Image layer data therefore does
NOT live under `/var/lib/docker` — it lives under `/var/lib/containerd`:
- `io.containerd.snapshotter.v1.overlayfs` — 8.0GB
- `io.containerd.content.v1.content` — 2.7GB

Together with ~245MB under `/var/lib/docker` (containers/rootfs/etc), that's
~11GB of the 16GB used on `/`, matching the unused images found via
`docker system df -v`. A plain `du -sh /var/lib/docker/*` undercounts because
(a) it's root-owned (needs sudo) and (b) the real data isn't even there —
check `/var/lib/containerd/*` instead. Also: `sudo du -sh /path/*` fails
silently if the `*` glob is expanded by your own shell before sudo runs
against a directory you can't list — wrap it as
`sudo bash -c 'du -sh /path/* | sort -rh'` to expand the glob as root.

## Fix
`docker image prune -a` — removes all images with zero attached containers.
Safe here since none of the unused wordpress:*/mariadb images had any
containers (running or stopped) referencing them.

## Workflow decision: where the prune step lives
Snapshots are currently taken manually via the DigitalOcean control panel
(no script/API automation yet). There's an existing `killsite` script that
tears down containers but does NOT prune images — and that's intentional,
not a gap to fix:

- `killsite` runs many times within a session as different WP/PHP version
  combos get spun up, torn down, and recreated. Pruning images on every
  `killsite` run would force re-pulling/rebuilding images each cycle,
  slowing down iteration.
- Instead, `docker image prune -a` should be run manually, once, right
  before taking a snapshot at the end of a session — not baked into
  `killsite`. Do not "helpfully" automate this into killsite later without
  re-confirming with the user.

The existing bloated snapshot won't shrink retroactively — delete it and
take a fresh one after pruning if a smaller snapshot is wanted on record.
