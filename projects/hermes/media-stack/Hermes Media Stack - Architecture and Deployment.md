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
| Additional media server | Plex | Separate running Compose project confirmed; configuration not supplied |
| TV automation | Sonarr | TCP 8989 |
| Film automation | Radarr | TCP 7878 |
| Indexer management | Prowlarr | Host port 9696; linked to Sonarr and Radarr |
| Downloading | SABnzbd | Media instance host port 8081 → container 8080; separate SABnzbd project exists |
| Media requests | Seerr | Confirmed `ghcr.io/seerr-team/seerr:latest` in the six-service media Compose project, host port 5055 |
| Other workloads | Scrypted, SillyTavern | Hosted alongside media services; not part of the core media acquisition path |

## Storage and network boundaries

The Pi and NAS devices are wired to a 1 Gbps LAN. Synology DS3 provides media and application data over NFS; DS2 was also used by a separate SABnzbd instance. Host mount paths recorded in earlier discussions include:

- `/mnt/ds3/media` — DS3 media files
- `/mnt/ds3/sillytavern` — separate application data
- `/mnt/ds2/main` — DS2 storage

The authoritative export strings and NAS IP addresses must be verified from live mount configuration; they are intentionally **not** asserted here.

The media library includes `/mnt/ds3/media/movies` and `/mnt/ds3/media/shows`. Application configuration was also stored locally under `/var/lib/media-stack/` for several services.

## Processing flow

1. Prowlarr manages indexers and propagates them to Sonarr/Radarr.
2. Sonarr and Radarr manage requested series and films and communicate with SABnzbd.
3. SABnzbd uses Pi SSD storage for incomplete work, with completed material destined for NAS storage.
4. Jellyfin serves NAS media; a separate running Plex Compose project is confirmed, but its library configuration is not yet supplied.

The exact completed-download path mappings, category names and import strategy must be read from live service configuration before reproducing the deployment.

## Verified outcomes and limitations

Jellyfin successfully played media on a TV; logs showed video stream copy and AAC audio transcoding. Sonarr, Radarr and Prowlarr integrations were tested after initial API/hostname issues. A later power outage exposed the dependency of running containers on healthy NFS mounts.

**Not established by available history:** a canonical end-to-end Compose stack, complete network definitions, current image digests, current Seerr configuration, or a tested clean-room rebuild. Preserve these as verification tasks rather than treating the diagram as an installation manifest.

## Dependency boundaries and ownership

The following responsibilities are distinct and should not be collapsed into a single end-to-end arrow:

```text
Prowlarr ──indexer configuration──> Sonarr / Radarr
Sonarr / Radarr ──download requests──> SABnzbd-media
SABnzbd-media ──completed download──> completed-download location (verify)
Sonarr / Radarr ──import/rename──> DS3 library (verify container path maps)
Jellyfin ──read/playback──> DS3 library
```

Cross-filesystem transfers can behave differently from a same-filesystem rename or hardlink. The original host/container path mapping and effective import operation must be established from service configuration, not inferred from the diagram.

## Authoritative configuration evidence to capture

For a reproducible build, collect the Portainer stack definitions or original Compose files, NAS mount units/`fstab` entries, the actual Docker volume targets and networks, and non-secret application settings. Keep live API keys and credentials out of the vault.

Related notes: [[Hermes Media Stack - Docker Compose and Portainer]], [[Hermes Media Stack - Synology NFS Storage]], [[Hermes Media Stack - SABnzbd Download Pipeline]], [[Hermes Media Stack - Operational Validation Runbook]].

## Verified 2026-10-18 media project configuration

The user supplied the actual six-service `media` Compose definition. All six services share a named bridge network `media`:

| Service | Image | Host → container port |
| --- | --- | --- |
| Jellyfin | `jellyfin/jellyfin:latest` | 8096 → 8096/tcp; 7359 → 7359/udp |
| SABnzbd-media | `lscr.io/linuxserver/sabnzbd:latest` | **8081 → 8080/tcp** |
| Sonarr | `lscr.io/linuxserver/sonarr:latest` | 8989 → 8989/tcp |
| Radarr | `lscr.io/linuxserver/radarr:latest` | 7878 → 7878/tcp |
| Prowlarr | `lscr.io/linuxserver/prowlarr:latest` | 9696 → 9696/tcp |
| Seerr | `ghcr.io/seerr-team/seerr:latest` | 5055 → 5055/tcp |

Portainer's Compose project inventory also confirms separate running `plex` and `sabnzbd` projects, plus `sillytavern`. These are **not** part of the six-service `media` Compose project.

See [[Hermes Media Stack - Docker Compose and Portainer]] for the supplied Compose definition and bind-mount table.
