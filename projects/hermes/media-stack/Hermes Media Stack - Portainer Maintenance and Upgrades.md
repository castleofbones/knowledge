---
type: how-to
status: active
created: 2026-10-18
source: ""
tags:
  - portainer
  - docker
  - container-upgrades
  - raspberry-pi
---

# Maintaining and Upgrading Hermes Docker Services with Portainer

## Context

Hermes uses Docker with Portainer CE for management. Portainer was exposed over HTTPS on TCP 9443. During October 2026 maintenance, the Portainer CE container was successfully updated using a stop/remove/pull/recreate sequence; the reported outcome was successful.

This note records the operational principles and verification steps. The original exact Portainer `docker run` command and complete volume/port flags were not available in the accessible record, so it does **not** invent them.

## Inspect before making changes

```bash
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
docker inspect portainer
docker logs --tail 100 portainer
```

The name `portainer` is illustrative until confirmed by `docker ps -a`. Record the live container's image reference, named volumes/binds, published ports, restart policy, socket mount and environment. Keep credentials and tokens out of documentation.

## Verified maintenance pattern

The previously successful high-level sequence was:

1. Inspect the existing container and preserve its exact recreation configuration.
2. Stop the container.
3. Remove the stopped container, **not its persistent data volume**.
4. Pull the intended image.
5. Recreate the container with equivalent persistent storage, ports and other settings.
6. Confirm HTTPS access and that existing stacks remain visible.

Do not run `docker rm -v` on a service with persistent data unless deliberately intending to delete its anonymous volumes. Prefer pinned image versions for reproducibility; historical use of `latest` does not identify a reproducible image.

## Post-upgrade checks

```bash
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
docker logs --tail 100 portainer
```

Confirm Portainer is reachable on its expected endpoint and can display the Hermes environment and stacks. Verify that Docker workloads were not unintentionally recreated or disconnected.

## NFS dependency

Before maintenance that restarts media containers, verify NAS mounts. A Docker restart during an NFS outage can attach containers to local underlay directories; recovering NFS on the host later does not necessarily correct their mount namespaces.

## Evidence limits

The historical upgrade succeeded, but exact command-line arguments, image digest and version change are not established in this note. Recover the authoritative deployment definition before repeating the operation.
