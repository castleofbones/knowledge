---
type: how-to
status: active
created: 2026-10-18
source: ""
tags:
  - plex
  - docker
  - media-server
  - synology-nas
---

# Deploying Plex Alongside Jellyfin on Hermes

## Purpose and verified deployment

Plex runs in a **separate Docker Compose project** from the six-service `media` stack on Hermes. The user supplied its actual Compose definition on 2026-10-18. It uses the host network, stores persistent configuration on the Pi, and reads movies and television shows from Synology DS3 through read-only bind mounts.

The Compose service definition establishes configuration, not a new playback acceptance test. Plex application-level library names, authentication, remote access and playback settings were not supplied.

## Verified Compose service

```yaml
services:
  plex:
    image: plexinc/pms-docker:latest
    container_name: plex

    network_mode: host

    environment:
      - TZ=Europe/London
      - PLEX_UID=1000
      - PLEX_GID=100

    volumes:
      - /var/lib/media-stack/plex/config:/config
      - /mnt/ds3/media/movies:/media/movies:ro
      - /mnt/ds3/media/shows:/media/shows:ro

    restart: unless-stopped
```

The `latest` image tag is floating, so this definition does not pin a reproducible Plex version. No `ports` stanza is needed or appropriate with `network_mode: host`.

## Storage and identity

| Purpose | Host path | Container path | Mode |
| --- | --- | --- | --- |
| Plex configuration | `/var/lib/media-stack/plex/config` | `/config` | Read/write |
| Movies | `/mnt/ds3/media/movies` | `/media/movies` | Read-only |
| Television shows | `/mnt/ds3/media/shows` | `/media/shows` | Read-only |

Plex is configured with `PLEX_UID=1000` and `PLEX_GID=100`. These are environment variables interpreted by the image, unlike Jellyfin's Compose-level `user: "1000:100"`.

Jellyfin uses the same two NAS-backed library paths as read-only mounts, but stores its own configuration and cache separately. This avoids the two services sharing their application metadata databases.

## Networking and lifecycle

Plex uses `network_mode: host`, whereas Jellyfin joins the named `media` bridge network. Plex's service definition contains no explicit port mappings; its listening ports depend on Plex's application behaviour. Both services use `restart: unless-stopped`.

## Validation

Before starting or recreating the Plex container, confirm the DS3 NFS mounts are present and contain the expected media files:

```bash
findmnt -T /mnt/ds3/media/movies -o TARGET,SOURCE,FSTYPE
findmnt -T /mnt/ds3/media/shows -o TARGET,SOURCE,FSTYPE
docker compose ls --all
docker ps --filter name=plex --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}'
docker inspect plex --format '{{json .Mounts}}'
docker logs --tail 100 plex
```

Validate the Plex UI and playback separately; neither is proven by Compose syntax alone. After a NAS outage, verify the running container's filesystem view before trusting that a host-side remount corrected existing binds.

## Remaining details

Capture the deployed image digest/version and actual Plex library settings if an exact point-in-time rebuild is required. The Compose service, storage mappings, identity, networking and restart policy are verified.

Related notes: [[Hermes Media Stack - Architecture and Deployment]], [[Hermes Media Stack - Jellyfin Deployment]], [[Hermes Media Stack - Synology NFS Storage]].
