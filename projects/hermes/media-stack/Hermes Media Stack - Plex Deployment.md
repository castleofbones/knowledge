---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - plex
  - docker
  - media-server
  - synology-nas
---

# Deploying Plex Alongside Jellyfin on Hermes

## What is established

Plex was discussed as a service running alongside Jellyfin on Hermes and was included in later media-recovery monitoring. Both media servers depend on the NAS-backed library, but the accessible record does **not** preserve the original Plex Compose service, its networking mode, ports, image, volume targets or library configuration.

This is therefore a **deployment evidence and validation note**, not a tested Plex installation guide.

## Coexistence design considerations

Two media servers can read the same film and series files without sharing their application databases. Before editing the deployment, establish whether both have read-only media bind mounts, whether each uses its own persistent configuration, and how their advertised endpoints avoid conflicts. These are checks to perform, **not verified historical settings**.

## Inspect the running service

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}'
docker inspect plex
docker logs --tail 100 plex
```

The `plex` container name is **not confirmed**. First obtain its real name with `docker ps -a` and substitute it in the commands.

Record the deployed image tag/digest, host network or bridge configuration, ports, environment, volumes, media paths and user IDs. Do not store Plex tokens or account credentials in the knowledge base.

## Validation checklist

1. Confirm Plex is running and its UI is reachable.
2. Inspect film and series library paths and map them to the expected DS3 NFS export.
3. Confirm playback on a supported client.
4. Verify that Plex and Jellyfin do not write into one another's metadata/configuration directories.
5. After NFS disruption, confirm Plex's mount namespace sees the actual NAS export.

## Outstanding work

Recover and validate the live Plex deployment definition, then replace this draft with an exact procedure. No Plex-specific performance or playback claims are made here.
