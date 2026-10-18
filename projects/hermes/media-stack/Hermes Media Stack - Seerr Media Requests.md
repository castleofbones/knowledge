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

# Media Requests on Hermes — Historical Overseerr Configuration

## Historical evidence and naming

A December 2025 configuration supplied in conversation explicitly describes **Overseerr**, not a verified Seerr deployment. The proposed article title in the documentation plan used Seerr, but the accessible evidence does not establish whether or when a migration occurred.

The original user-supplied Compose excerpt was (retained as historical evidence, **not** an executable deployment template):

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

**Historical source note:** This reflects a user-supplied configuration, not a current verified deployment. Compose combines `network_mode: host` and `ports` here; Host networking and published port mappings conflict: a Compose implementation should use host networking **without** `ports`, or bridge networking with the appropriate `ports` mapping, according to the live deployment. Neither correction is verified as the historical running configuration; do not deploy this example unchanged. The historical process was observed listening on TCP 5055, and `ufw` was not installed at the time.

## What to inspect now

```bash
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}'
ss -tulpn | grep 5055
```

If an Overseerr or Seerr service is present, retrieve its **current** Compose/Portainer configuration, authentication model, application data volume, media-server connection and Sonarr/Radarr integration. Do not record an unverified product migration as completed.

## Validation

Verify network reachability from another LAN device, then test authorised request submission and the intended approval/automation flow. Do not treat a listening socket as proof of end-to-end request processing.

## Outstanding work

Confirm the currently deployed product (Overseerr, Seerr or neither), its actual network mode and its media automation integrations. Until then, this note remains draft rather than claiming a completed Seerr build.
