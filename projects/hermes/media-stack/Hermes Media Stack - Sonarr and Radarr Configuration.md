---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - sonarr
  - radarr
  - sabnzbd
  - media-library
---

# Configuring Sonarr and Radarr Library Management

## Implemented services

Sonarr manages television series on Hermes; Radarr manages films. The verified ports are **8989** (Sonarr) and **7878** (Radarr). Configuration data was placed under `/var/lib/media-stack/sonarr` and `/var/lib/media-stack/radarr`, with ownership conventions observed as UID 1000/GID 100.

The NAS media library uses:

```text
/mnt/ds3/media/
├── movies/
└── shows/
```

The original build populated Radarr's movies and Sonarr's shows. Historical counts were approximately 341 film directories and 67 series directories at one point; these are snapshots, not current inventory.

## External integrations

- Prowlarr linked to both applications for indexer management.
- SABnzbd-media supplied download handling.
- Sonarr/Radarr connectivity to SABnzbd initially failed with HTTP 403; API key and hostname-allowance corrections resolved the tests.

## Configuration and validation procedure

1. Confirm each application's **root folders** map to the intended container paths backed by DS3.
2. Confirm Docker volume mappings expose those folders consistently.
3. Inspect download client host, port, API key (without copying the secret) and categories.
4. Test the download-client connection.
5. Confirm Prowlarr-managed indexers are present and test them.
6. Verify imported titles point to existing media paths.
7. Inspect a representative item and verify files are accessible to the media server.

```bash
docker logs --tail 100 sonarr
docker logs --tail 100 radarr
docker inspect sonarr
docker inspect radarr
```

## Observed issues

Initially, poster artwork was missing despite libraries being populated. Connectivity to `skyhook.sonarr.tv` and `api.radarr.video` was tested and returned HTTP 200. This narrows the fault but does not by itself prove the precise artwork root cause; no final cause was established in the available record.

## Boundaries

Do not infer exact container root-folder paths from host paths; Docker bind destinations must be read from the actual deployment. Quality profiles, naming formats, retention settings and categories are not available as verified values.

## Outcome

Both applications were populated, Prowlarr was linked, and download-client connection tests passed. The record supports operational integration, not a complete clean-install recipe.

## Inspect actual paths and import configuration

```bash
docker inspect sonarr --format '{{json .Mounts}}'
docker inspect radarr --format '{{json .Mounts}}'
docker inspect sabnzbd-media --format '{{json .Mounts}}'
```

Host-side DS3 library paths are established, but the **container-side root folders** must be obtained from Sonarr/Radarr and compared to those mount destinations. Record the non-secret download client endpoint, categories, completed-download settings and any Remote Path Mappings.

A populated library is not an end-to-end import test: verify that a newly completed authorised download appears in the correct library and its file is visible to Jellyfin. An SSD-to-NAS handoff may cross filesystem boundaries, so don't assume hardlinks or atomic renames.

See [[Hermes Media Stack - SABnzbd Download Pipeline]] and [[Hermes Media Stack - Prowlarr Integration]].
