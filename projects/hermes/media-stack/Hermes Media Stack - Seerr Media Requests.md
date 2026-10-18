---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - seerr
  - overseerr
  - docker
  - media-requests
---

# Deploying Seerr for Media Requests on Hermes

## Historical evidence and naming

A December 2025 Hermes-era configuration explicitly deployed **Overseerr**, not a verified Seerr deployment. The proposed article title in the documentation plan used Seerr, but the accessible evidence does not establish whether or when a migration occurred.

The historical Compose excerpt was:

```yaml
services:
  overseerr:
    image: lscr.io/linuxserver/overseerr:latest
    container_name: overseerr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - overseerr_data:/config
    ports:
      - 5055:5055
    restart: unless-stopped
    network_mode: host

volumes:
  overseerr_data:
    external: true
```

**Historical source note:** This reflects a user-supplied configuration, not a current verified deployment. Compose combines `network_mode: host` and `ports` here; Docker Compose does not permit published port mappings with host networking, so this example should not be deployed unchanged. The historical process was observed listening on TCP 5055, and `ufw` was not installed at the time.

## What to inspect now

```bash
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}'
ss -tulpn | grep 5055
```

If an Overseerr or Seerr service is present, retrieve its **current** Compose/Portainer configuration, authentication model, application data volume, media-server connection and Sonarr/Radarr integration.

## Validation

Verify network reachability from another LAN device, then test authorised request submission and the intended approval/automation flow. Do not treat a listening socket as proof of end-to-end request processing.

## Outstanding work

Confirm the currently deployed product (Overseerr, Seerr or neither), its actual network mode and its media automation integrations. Until then, this note remains draft rather than claiming a completed Seerr build.
