---
type: how-to
status: active
created: 2026-10-18
source: ""
tags:
  - seerr
  - overseerr
  - docker
  - media-requests
---

# Seerr on Hermes — Confirmed Deployment and Historical Overseerr Context

## Historical evidence and naming

The user supplied a **current Seerr Compose service** on 2026-10-18. A December 2025 conversation also contained a historical Overseerr Compose example. Both facts are recorded, but the date and procedure of any migration between the two products are not established.

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

The current `media` Compose definition confirms Seerr. Its non-secret media-server/request destinations are now verified below; the operator-confirmed approval policy is documented below; any historical Overseerr migration remains unverified.

## Current confirmed Seerr Compose service (2026-10-18)

```yaml
seerr:
  image: ghcr.io/seerr-team/seerr:latest
  container_name: seerr
  init: true
  environment:
    - TZ=Europe/London
    - PORT=5055
  ports:
    - "5055:5055"
  volumes:
    - /var/lib/media-stack/seerr/config:/app/config
  healthcheck:
    test: wget --no-verbose --tries=1 --spider http://localhost:5055/api/v1/settings/public || exit 1
    start_period: 20s
    timeout: 3s
    interval: 15s
    retries: 3
  security_opt:
    - no-new-privileges
  cap_drop:
    - ALL
  restart: unless-stopped
  networks:
    - media
```

The service joins the named `media` bridge network, exposes TCP 5055, persists application data under `/var/lib/media-stack/seerr/config`, and includes an HTTP health check plus `no-new-privileges` and dropped Linux capabilities. The Compose definition establishes deployment intent and structure; it does not independently establish that a particular request/approval workflow passed an end-to-end test.

## Verified live request integrations (2026-10-18)

The following non-secret settings were extracted from Seerr's persistent `settings.json`:

| Service | Recorded configuration |
| --- | --- |
| Jellyfin | Name `jellyfin`, port `8096`, SSL disabled |
| Plex | Empty name, port `32400`, SSL disabled; **not evidence of a functioning Plex integration** |
| Sonarr | Name `Sonarr`, host `sonarr`, port `8989`, SSL disabled, default server, profile ID `6`, root `/data/shows` |
| Radarr | Name `Radarr`, host `radarr`, port `7878`, SSL disabled, default server, profile ID `6`, root `/data/movies` |

The `main` section yielded no fields under the extraction allowlist; this is not evidence that Seerr has no main settings. These values establish configured request destinations, **not** approval rules, automatic-request policy, or successful end-to-end requests. The historical Overseerr migration remains unverified.

## Request approval policy (operator confirmed, 2026-10-18)

The operator confirms that **their own requests are automatically approved**, while **requests from other users require the operator's approval** before proceeding. This is the configured intended approval workflow in Seerr, rather than an inferred default. This statement does not assert that each permission was independently tested with separate user accounts. No user identifiers or credentials are recorded here.
