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

Do not infer exact container root-folder paths from host paths; Docker bind destinations must be read from the actual deployment. Naming formats and completed-download removal are now verified below; quality-profile definitions and categories remain unverified.

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

## Confirmed Docker mounts (2026-10-18)

Both services use `lscr.io/linuxserver/*:latest`, UID 1000, GID 100 and `TZ=Europe/London`:

```yaml
sonarr:
  ports: ["8989:8989"]
  volumes:
    - /var/lib/media-stack/sonarr/config:/config
    - /mnt/ds3/media:/data
radarr:
  ports: ["7878:7878"]
  volumes:
    - /var/lib/media-stack/radarr/config:/config
    - /mnt/ds3/media:/data
```

The YAML excerpt intentionally shows the relevant mappings only, **not** full standalone service definitions. The complete deployment is in [[Hermes Media Stack - Docker Compose and Portainer]]. Inside these containers, the NAS-backed paths are available at `/data/movies`, `/data/shows`, and `/data/downloads`. The actual root-folder and completed-download configuration in each application's UI is still a separate verification step.

## Verified application settings (2026-10-18)

These values were extracted from consistent read-only snapshots of the live application SQLite databases, without exporting API keys.

| Setting | Sonarr | Radarr |
| --- | --- | --- |
| Root folder | `/data/shows/` | `/data/movies/` |
| Download client | `SABnzbd Media` (enabled) | `SABnzbd Media` (enabled) |
| Download client endpoint | `sabnzbd-media:8080`, SSL disabled | `sabnzbd-media:8080`, SSL disabled |
| Remove completed downloads | Enabled (`1`) | Enabled (`1`) |
| Remote path mappings | None | None |
| Synced indexer | `NzbPlanet (Prowlarr)` (Newznab, RSS enabled) | `NzbPlanet (Prowlarr)` (Newznab, RSS enabled) |
| Automatic renaming | **Disabled** (`RenameEpisodes=0`) | **Disabled** (`RenameMovies=0`) |

The naming templates are configured but **not currently used for automatic renaming**:

- Sonarr episode: `{Series Title} - S{season:00}E{episode:00} - {Episode Title} {Quality Full}`; season folder: `Season {season}`.
- Radarr movie: `{Movie Title} ({Release Year}) {Quality Full}`; movie folder: `{Movie Title} ({Release Year})`.

Both services bind DS3 `/mnt/ds3/media` to container `/data`, so their root folders and SABnzbd's completed-download location share the same container path namespace. Category names, additional download-client settings, quality-profile definitions and the precise category definitions remain outside the extracted evidence.

## Operational evidence — recent successful imports (user confirmation)

The operator confirms that numerous new movies and television shows have been added successfully through the running stack recently. This is evidence of the download-to-library workflow working in normal operation, not merely passing an API connectivity test. A separate synthetic end-to-end download is therefore **not required to establish that the current pipeline works**. The exact SABnzbd category definitions and Seerr approval policy remain undocumented; their absence from this record does not imply a malfunction.

## SABnzbd category destinations (verified in application UI)

SABnzbd has a `movies` category writing to `/data/downloads/movies` and a `shows` category writing to `/data/downloads/shows`. These are the configured SABnzbd category destinations; the final library roots are Radarr `/data/movies/` and Sonarr `/data/shows/`. The screenshot alone does not independently prove which category string each download client sends, so do not conflate the existence of a category with verification of each client's category assignment.
