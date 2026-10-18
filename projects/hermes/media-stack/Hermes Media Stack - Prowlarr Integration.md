---
type: how-to
status: active
created: 2026-10-18
source: ""
tags:
  - prowlarr
  - sonarr
  - radarr
  - indexers
---

# Configuring Prowlarr Indexers and Sonarr/Radarr Integration

## Purpose

Prowlarr centralises indexer management for the Hermes media automation stack. It was deployed and linked to both Sonarr and Radarr during the October 2026 build. The integrations were subsequently reported as passing tests.

## Verified architecture

```text
           Prowlarr
           /      \
        Sonarr   Radarr
           \      /
          SABnzbd-media
               |
         NAS media library
```

Prowlarr supplies indexer configuration to Sonarr/Radarr. Sonarr and Radarr independently manage download clients and library imports. Prowlarr is **not** the download client.

## Reproduce the integration from live configuration

1. Open Prowlarr and inspect configured indexers.
2. Under its application integrations, confirm entries for Sonarr and Radarr.
3. Record the internal service URLs and synchronisation settings; omit API keys.
4. Run Prowlarr's connection tests.
5. In Sonarr and Radarr, confirm indexers are present and enabled as intended.
6. Test a permitted search and inspect application logs if results are missing.

Relevant container diagnostics:

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}'
docker logs --tail 100 sonarr
docker logs --tail 100 radarr
```

The exact Prowlarr port, Docker network name, indexer list and application URL fields were not captured in the accessible history.

## Integration lessons

A passing service-to-service API test is stronger evidence than a UI that merely loads. Keep the roles separate when troubleshooting: missing search results may originate with an indexer, Prowlarr synchronisation, Sonarr/Radarr restrictions, or the download client.

## Outcome

Prowlarr was added and linked to Sonarr/Radarr; the media automation tests passed. This article records the integration pattern and verification points, not a claim that every indexer or future search will work indefinitely.
