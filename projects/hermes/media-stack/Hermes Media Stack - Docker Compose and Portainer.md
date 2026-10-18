---
type: how-to
status: active
created: 2026-10-18
source: ""
tags:
  - docker-compose
  - portainer
  - raspberry-pi
  - container-management
---

# Hermes Docker and Portainer Deployment — Known Configuration and Reconstruction

## Scope

Hermes runs Docker services on a Raspberry Pi 5 under Portainer CE. This article captures verified deployment conventions and a **reconstruction checklist**. The `media` project's actual Compose definition was supplied on 2026-10-18 and is recorded below. Other projects' Compose definitions remain outstanding.

## Prerequisites

- Debian 13 (Trixie) ARM64 on Raspberry Pi 5.
- Working Docker Engine and Portainer CE; Portainer was available over HTTPS on TCP 9443.
- Synology NFS exports mounted and accessible **before** starting dependent services.
- Appropriate bind-mount directories and UID/GID ownership.

## Verified service inventory

| Service | Observed port | Notes |
| --- | --- | --- |
| Jellyfin | 8096/tcp, 7359/udp | Healthy and playback verified |
| Sonarr | 8989/tcp | Application configuration under `/var/lib/media-stack/sonarr` |
| Radarr | 7878/tcp | Application configuration under `/var/lib/media-stack/radarr` |
| SABnzbd media | 8081/tcp host → 8080/tcp container | Separate SABnzbd project also exists |
| Portainer | 9443/tcp | Management UI |
| Plex | Separate running Compose project | Compose definition not yet supplied |
| Prowlarr | 9696/tcp | Linked to Sonarr/Radarr |
| Seerr | 5055/tcp | Confirmed in media Compose definition |

Several media-stack directories used `uid=1000`, `gid=100`. Do not assume those IDs apply to every container; inspect the live definitions.

## Reconstructing the authoritative deployment

Inspect the actual containers and save the results before editing:

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}'
docker inspect jellyfin
docker inspect sonarr
docker inspect radarr
docker inspect sabnzbd-media
```

For each service, record its image and version, restart policy, environment variables (redact secrets), ports, networks, bind sources and destinations, user identity, and health check. Export or retrieve the Compose/Portainer stack definition from its actual deployment source. Do not reverse-engineer credentials from an inspection dump into the knowledge base.

## Deployment order and validation

1. Confirm NFS availability and the intended NAS files are visible at every host mount.
2. Confirm configuration directories exist and ownership is correct.
3. Deploy or start supporting services, then media applications.
4. Check `docker ps`, container logs and each application UI.
5. Verify Sonarr/Radarr-to-SABnzbd API tests and Prowlarr indexer propagation.
6. Confirm Jellyfin can browse and play actual NAS media.

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}'
docker logs --tail 100 sonarr
docker logs --tail 100 radarr
docker logs --tail 100 jellyfin
```

## Safety notes

A successful host-side NFS mount does **not** prove that an already-running container sees that mount. After outages, inspect container mount namespaces before trusting service health. Avoid placing application data on a NAS path that might silently resolve to the Pi SSD underlay.

## Outstanding evidence

Recover the original Compose files and current Portainer stack exports to convert this from a documented reconstruction procedure into a fully reproducible, tested deployment runbook.

## Inspect Compose provenance, rather than guessing a manifest

Docker Compose-managed containers often carry the labels `com.docker.compose.project`, `com.docker.compose.service`, and `com.docker.compose.project.config_files`. Query these labels to locate the actual source where available:

```bash
docker ps -a --format 'table {{.Names}}\t{{.Label "com.docker.compose.project"}}\t{{.Label "com.docker.compose.service"}}'
docker inspect sonarr --format '{{json .Config.Labels}}'
docker inspect sonarr --format '{{json .Mounts}}'
docker inspect sonarr --format '{{json .NetworkSettings.Networks}}'
```

Absence of Compose labels may mean a service was created directly by Portainer or Docker rather than by Compose. Even when labels exist, they do not guarantee that the source file remains available. A verified Compose definition should include exact images, UID/GID, restart policies, mapped ports, named networks, config/data bindings, health checks and dependency behaviour.

**Do not publish** raw `docker inspect` dumps containing tokens, passwords, endpoints or other secrets. Sanitise before committing to the vault.

See [[Hermes Media Stack - Architecture and Deployment]] for component roles and [[Hermes Media Stack - Operational Validation Runbook]] for post-deploy acceptance.

## Verified media stack Compose definition — supplied 2026-10-18

The following is a compact, semantically equivalent transcription of the **user-supplied** Portainer Compose definition. It is not an independently redeployed test. It uses floating `latest` image tags, so a future pull can differ from the recorded deployment.

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    user: "1000:100"
    ports: ["8096:8096", "7359:7359/udp"]
    volumes:
      - /var/lib/media-stack/jellyfin/config:/config
      - /var/lib/media-stack/jellyfin/cache:/cache
      - /mnt/ds3/media/movies:/media/movies:ro
      - /mnt/ds3/media/shows:/media/shows:ro
    restart: unless-stopped
    networks: [media]
  sabnzbd-media:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzbd-media
    environment: [PUID=1000, PGID=100, TZ=Europe/London]
    ports: ["8081:8080"]
    volumes:
      - /var/lib/media-stack/sabnzbd/config:/config
      - /var/lib/media-stack/sabnzbd/incomplete:/incomplete-downloads
      - /mnt/ds3/media/downloads:/data/downloads
    restart: unless-stopped
    networks: [media]
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment: [PUID=1000, PGID=100, TZ=Europe/London]
    ports: ["8989:8989"]
    volumes:
      - /var/lib/media-stack/sonarr/config:/config
      - /mnt/ds3/media:/data
    restart: unless-stopped
    networks: [media]
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment: [PUID=1000, PGID=100, TZ=Europe/London]
    ports: ["7878:7878"]
    volumes:
      - /var/lib/media-stack/radarr/config:/config
      - /mnt/ds3/media:/data
    restart: unless-stopped
    networks: [media]
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment: [PUID=1000, PGID=100, TZ=Europe/London]
    ports: ["9696:9696"]
    volumes:
      - /var/lib/media-stack/prowlarr/config:/config
    restart: unless-stopped
    networks: [media]
  seerr:
    image: ghcr.io/seerr-team/seerr:latest
    container_name: seerr
    init: true
    environment: [TZ=Europe/London, PORT=5055]
    ports: ["5055:5055"]
    volumes:
      - /var/lib/media-stack/seerr/config:/app/config
    healthcheck:
      test: wget --no-verbose --tries=1 --spider http://localhost:5055/api/v1/settings/public || exit 1
      start_period: 20s
      timeout: 3s
      interval: 15s
      retries: 3
    security_opt: [no-new-privileges]
    cap_drop: [ALL]
    restart: unless-stopped
    networks: [media]
networks:
  media:
    name: media
    driver: bridge
```

### Actual project location and related stacks

The supplied `docker compose ls --all` output reports the running `media` project with **six services** and configuration at `/data/compose/6/docker-compose.yml`. Separate projects exist for `plex` (`/data/compose/7/docker-compose.yml`), `sabnzbd` (`/data/compose/4/docker-compose.yml`), and `sillytavern` (`/data/compose/1/docker-compose.yml`). Those other Compose definitions have **not** been supplied; do not assume they share the `media` network.

### Actual host-to-container storage map

| Service | Bind mounts |
| --- | --- |
| Jellyfin | `/var/lib/media-stack/jellyfin/config:/config`; `/var/lib/media-stack/jellyfin/cache:/cache`; DS3 movies → `/media/movies:ro`; DS3 shows → `/media/shows:ro` |
| SABnzbd-media | `/var/lib/media-stack/sabnzbd/config:/config`; `/var/lib/media-stack/sabnzbd/incomplete:/incomplete-downloads`; `/mnt/ds3/media/downloads:/data/downloads` |
| Sonarr | `/var/lib/media-stack/sonarr/config:/config`; `/mnt/ds3/media:/data` |
| Radarr | `/var/lib/media-stack/radarr/config:/config`; `/mnt/ds3/media:/data` |
| Prowlarr | `/var/lib/media-stack/prowlarr/config:/config` |
| Seerr | `/var/lib/media-stack/seerr/config:/app/config` |

The `media` network is a named Docker bridge network. For SABnzbd, `8081` is the **host** port; `8080` is the **container** port. Other containers on the same network should use the container's service name and container port where configured. See [[Hermes Media Stack - SABnzbd Download Pipeline]].
