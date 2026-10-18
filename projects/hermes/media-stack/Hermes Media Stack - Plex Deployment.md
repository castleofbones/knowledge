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

Plex coexistence was mentioned in prior discussion, but the available build evidence does **not** establish that Plex is currently deployed on Hermes. The original Plex Compose service, networking, ports, image, bind mounts and application configuration were not recovered. No NAS dependency should be assumed until a running Plex service and its mounts have been inspected.

This is therefore a **deployment evidence and validation note**, not a tested Plex installation guide.

## Coexistence design considerations

**If Plex is deployed**, two media servers can read the same film and series files without sharing their application databases. Before editing the deployment, establish whether both have read-only media bind mounts, whether each uses its own persistent configuration, and how their advertised endpoints avoid conflicts. These are checks to perform, **not verified historical settings**.

## Inspect the running service

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}'
docker inspect plex
docker logs --tail 100 plex
```

The `plex` container name is **not confirmed**, and neither is deployment. First obtain its real name (if any) with `docker ps -a`; run the subsequent inspect/log commands only if that container exists.

Record the deployed image tag/digest, host network or bridge configuration, ports, environment, volumes, media paths and user IDs. Do not store Plex tokens or account credentials in the knowledge base.

## Validation checklist

1. Determine whether Plex is deployed at all; only then confirm its UI is reachable.
2. Inspect film and series library paths and map them to the expected DS3 NFS export.
3. Confirm playback on a supported client.
4. Verify that Plex and Jellyfin do not write into one another's metadata/configuration directories.
5. After NFS disruption, confirm Plex's mount namespace sees the actual NAS export.

## Outstanding work

Recover and validate the live Plex deployment definition, then replace this draft with an exact procedure. No Plex-specific performance or playback claims are made here.
