---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - media-server
  - docker
  - nfs
  - operations
---

# Hermes Media Stack — Validation and Operational Runbook

## Purpose

This runbook checks whether the deployed media stack is functioning after routine maintenance, a restart or an outage. It is based on the known Hermes service topology; it has **not** been executed as a single complete acceptance suite.

## 1. Host and NAS mounts

```bash
uname -a
docker version
findmnt -T /mnt/ds3/media -o TARGET,SOURCE,FSTYPE
findmnt -T /mnt/ds2/main -o TARGET,SOURCE,FSTYPE
ls -ld /mnt/ds3/media/movies /mnt/ds3/media/shows
```

Expected DS3 media source: `10.10.0.252:/volume1/media`. Expected DS2 main source: `10.10.0.253:/volume1/main`. Do not continue to write-dependent service tests if the NAS mount is absent.

## 2. Container inventory and health

```bash
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
docker logs --tail 100 jellyfin
docker logs --tail 100 sonarr
docker logs --tail 100 radarr
```

Known service roles include Jellyfin, Sonarr, Radarr, Prowlarr, SABnzbd-media, Plex and Portainer. Confirm actual container names and deployed services rather than assuming every historical service still exists.

## 3. Media acquisition integration

- Confirm Prowlarr application links and indexer tests.
- Confirm Sonarr and Radarr download-client tests.
- Confirm SABnzbd API authentication and hostname allowance.
- Confirm intended incomplete download storage is on the Pi SSD and completed material goes to NAS storage.
- Check one existing library item in Sonarr and Radarr against the actual media path.

## 4. Playback

Play representative media in Jellyfin on the intended client. Check logs for direct playback versus transcoding. The historical successful TV playback copied video and transcoded audio to AAC.

## 5. Outage-specific checks

A host-side remount does not update a stale Docker mount namespace automatically. If the NFS recovery utility flags stale attachments, investigate the affected containers and any files written to local mount underlays **before** considering restarts.

The `media-recovery` systemd timer was later configured to run approximately once a minute, recover NFS automounts and start stopped containers. It deliberately does not restart running containers with stale mounts.

## Acceptance record

Capture date, container image versions, NFS sources, application test outcomes, playback outcome and unresolved deviations. Do not mark this runbook fully verified until the complete sequence has been performed on the current stack.
