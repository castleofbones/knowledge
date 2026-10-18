---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - docker-compose
  - portainer
  - raspberry-pi
  - container-management
---

# Deploying the Hermes Media Stack with Docker Compose and Portainer

## Scope

Hermes runs Docker services on a Raspberry Pi 5 under Portainer CE. This article captures verified deployment conventions and a **reconstruction checklist**. The original complete Compose manifests were not recovered, so no invented production-ready YAML is provided.

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
| SABnzbd media | 8080/tcp | Separate SABnzbd instance also existed |
| Portainer | 9443/tcp | Management UI |
| Plex | Not recovered | Later included in recovery monitoring |
| Prowlarr | Not recovered | Linked to Sonarr/Radarr |
| Seerr | Not verified | Historical Overseerr configuration used port 5055 |

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
