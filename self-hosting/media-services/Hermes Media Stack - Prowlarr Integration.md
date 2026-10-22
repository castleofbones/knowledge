---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - prowlarr
  - sonarr
  - radarr
  - indexers
purpose: knowledge
---

# Prowlarr Integration with Sonarr and Radarr on Hermes

## Purpose

Prowlarr centralises indexer management for the Hermes media automation stack. It was deployed and linked to both Sonarr and Radarr during the October 2026 build. The integrations were subsequently reported as passing tests.

## Application responsibilities

```text
           Prowlarr
           /      \
        Sonarr   Radarr
           \      /
       SABnzbd-media
           |
      completed files
           |
     Sonarr / Radarr import
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

The Compose definition confirms Prowlarr host port **9696** and the shared `media` bridge network. The live application URLs and configured indexer are recorded below.

## Integration lessons

A passing service-to-service API test is stronger evidence than a UI that merely loads. Keep the roles separate when troubleshooting: missing search results may originate with an indexer, Prowlarr synchronisation, Sonarr/Radarr restrictions, or the download client.

## Outcome

Prowlarr was added and linked to Sonarr/Radarr; the media automation tests passed. This article records the integration pattern and verification points, not a claim that every indexer or future search will work indefinitely.

## Connection fields to preserve when available

For each Prowlarr application integration, record its **non-secret** endpoint URL, sync profile, categories and test result. For each Sonarr/Radarr download client, separately record the host/port, category and tested status. API keys belong in the live secret/configuration store, not the vault.

See [[Hermes Media Stack - SABnzbd Download Pipeline]] for the storage split and [[Hermes Media Stack - Sonarr and Radarr Configuration]] for root-folder/import ownership.

## Confirmed deployment service

The 2026-10-18 `media` Compose definition deploys `lscr.io/linuxserver/prowlarr:latest` as `prowlarr`, exposes **9696:9696**, stores configuration at `/var/lib/media-stack/prowlarr/config:/config`, sets PUID 1000, PGID 100 and Europe/London timezone, and joins the shared `media` bridge network. This confirms the container's networking and persistence; application API URLs, indexer definitions and sync settings must still be read from Prowlarr itself.

## Verified live application integrations (2026-10-18)

A consistent read-only snapshot of `prowlarr.db` confirms the following configured applications and indexer (credentials omitted):

| Integration | Value |
| --- | --- |
| Sonarr application | `http://sonarr:8989`, `SyncLevel=2` |
| Radarr application | `http://radarr:7878`, `SyncLevel=2` |
| Enabled indexer | `NzbPlanet` (`Newznab`, `AppProfileId=1`) |
| Sync profile | `Standard` |
| Standard profile RSS | Enabled |
| Standard profile interactive search | Enabled |
| Standard profile automatic search | Enabled |

Sonarr and Radarr each have a synced `NzbPlanet (Prowlarr)` Newznab indexer with RSS enabled. The numeric `SyncLevel=2` is preserved as observed rather than assigning an unverified UI label. These records establish configured integrations; they do not prove that a new search succeeds at this moment.
