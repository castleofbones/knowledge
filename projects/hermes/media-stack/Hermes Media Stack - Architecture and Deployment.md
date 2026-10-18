---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - raspberry-pi
  - docker
  - media-server
  - synology-nas
---

# Hermes Media Stack — Architecture and Deployment

## Purpose and deployment status

Hermes is a Raspberry Pi 5 (16 GB) running Debian 13 (Trixie) and hosting a containerised media platform managed through Docker and Portainer CE. The media library lives primarily on Synology NAS storage rather than the Pi's local disk. This is a record of the implemented architecture as observed in the October 2026 build discussions, not a substitute for the live Compose specifications.

## Components

| Role | Component | Verified detail |
| --- | --- | --- |
| Container management | Portainer CE | HTTPS UI on port 9443 |
| Media playback | Jellyfin | TCP 8096; UDP 7359; TV playback verified |
| Additional media server | Plex | Included in subsequent recovery monitoring; detailed deployment configuration not recovered |
| TV automation | Sonarr | TCP 8989 |
| Film automation | Radarr | TCP 7878 |
| Indexer management | Prowlarr | Linked to Sonarr and Radarr |
| Downloading | SABnzbd | Media instance on TCP 8080; separate existing instance also present |
| Media requests | Seerr/Overseerr | Historical Overseerr service used port 5055; current Seerr deployment not independently verified |
| Other workloads | Scrypted, SillyTavern | Hosted alongside media services; not part of the core media acquisition path |

## Storage and network boundaries

The Pi and NAS devices are wired to a 1 Gbps LAN. Synology DS3 provides media and application data over NFS; DS2 provides a separate existing SABnzbd storage path. The recovery implementation later monitored:

- `/mnt/ds3/media` — `10.10.0.252:/volume1/media`
- `/mnt/ds3/sillytavern` — `10.10.0.252:/volume1/sillytavern`
- `/mnt/ds2/main` — `10.10.0.253:/volume1/main`

The media library includes `/mnt/ds3/media/movies` and `/mnt/ds3/media/shows`. Application configuration was also stored locally under `/var/lib/media-stack/` for several services.

## Processing flow

1. Prowlarr manages indexers and propagates them to Sonarr/Radarr.
2. Sonarr and Radarr manage requested series and films and communicate with SABnzbd.
3. SABnzbd uses Pi SSD storage for incomplete work, with completed material destined for NAS storage.
4. Jellyfin, and separately Plex, serve media from the NAS library.

The exact completed-download path mappings, category names and import strategy must be read from live service configuration before reproducing the deployment.

## Verified outcomes and limitations

Jellyfin successfully played media on a TV; logs showed video stream copy and AAC audio transcoding. Sonarr, Radarr and Prowlarr integrations were tested after initial API/hostname issues. A later power outage exposed the dependency of running containers on healthy NFS mounts.

**Not established by available history:** a canonical end-to-end Compose stack, complete network definitions, current image digests, current Seerr configuration, or a tested clean-room rebuild. Preserve these as verification tasks rather than treating the diagram as an installation manifest.
