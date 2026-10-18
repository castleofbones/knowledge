---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - sabnzbd
  - download-automation
  - ssd
  - synology-nas
---

# Setting Up SABnzbd with SSD-Based Incomplete Downloads

## Architectural decision

The documented design intention was that **incomplete downloads and repair work stay on the Raspberry Pi SSD**, while completed downloads move to NAS storage. The Docker bind paths have now been recovered: incomplete staging is mapped from the Pi SSD, and `/data/downloads` maps to DS3. The SABnzbd **Folders** selections are now confirmed from its live INI; end-to-end import behaviour still requires confirmation. This avoids performing temporary high-churn work directly on the network share.

The system also retained an existing, separate SABnzbd instance associated with DS2. The newer media-stack instance is named `sabnzbd-media`, published on **host TCP 8081** and listening on **container TCP 8080**.

## Known implementation facts

- Pi SSD: staging location for incomplete work.
- Synology NAS: intended destination for completed work.
- Media library on DS3: `/mnt/ds3/media/movies` and `/mnt/ds3/media/shows`.
- DS2: existing SABnzbd storage context at `/mnt/ds2/main`.
- Sonarr and Radarr connect to the media SABnzbd instance through its API.
- During integration, the request from Sonarr initially received HTTP 403. Correcting the API key and SABnzbd hostname allowance resolved the issue.

The Docker volume targets are now documented below. The exact SABnzbd application folder selections, categories and final import semantics must still be read from the deployed configuration.

## Revalidate the pipeline

1. In SABnzbd, inspect **Folders** for incomplete and completed paths.
2. Inspect **Categories** used by Sonarr and Radarr; record their exact names and any folder overrides.
3. Check Docker bind mounts to establish where each container path resolves on the host.
4. Verify the API key and hostname allowlist without recording the secret.
5. Test the SABnzbd connection from Sonarr and Radarr.
6. Perform a small, authorised end-to-end download/import test and verify that the finished media appears on DS3, not only in the download folder.

Container-side checks:

```bash
docker inspect sabnzbd-media
docker logs --tail 100 sabnzbd-media
docker inspect sonarr
docker inspect radarr
```

## Common integration fault: HTTP 403

The initial symptom was a 403 response when testing connectivity to `sabnzbd-media`, including from inside Sonarr. This did not establish a network outage: the HTTP service was reachable and rejecting the request. The resolution involved the correct API key and permitted hostname.

## Operational caveats

Do not assume that two SABnzbd instances share API keys, download folders or categories. Avoid storing keys in Obsidian. Before moving download paths, verify that completed-download handling and Sonarr/Radarr imports still work.

**Unverified:** the exact final category mapping, atomic-move versus copy behaviour across filesystems, and a tested full installation procedure.

## Path map required before a reproducible rebuild

Keep these independently verified values together. Do **not** assume that a host path is also the container path.

| Stage | Host path/filesystem | Container path | Owner |
| --- | --- | --- | --- |
| Incomplete download | Inspect Pi SSD directory | Inspect SABnzbd Folders | SABnzbd |
| Completed download | Inspect target volume | Inspect SABnzbd Folders | SABnzbd |
| Series library | `/mnt/ds3/media/shows` | Inspect Sonarr Root Folder | Sonarr |
| Film library | `/mnt/ds3/media/movies` | Inspect Radarr Root Folder | Radarr |

To identify the actual bind targets without dumping application secrets:

```bash
docker inspect sabnzbd-media --format '{{json .Mounts}}'
docker inspect sonarr --format '{{json .Mounts}}'
docker inspect radarr --format '{{json .Mounts}}'
```

Check **Completed Download Handling**, categories, and any **Remote Path Mappings** configured in Sonarr/Radarr. The latter are sometimes necessary when a download client reports a container path that its consumers cannot see, but their use on Hermes has not been established.

A same-filesystem atomic rename or hardlink is not generally available between the Pi SSD and a NAS filesystem. Determine whether the configured process copies, moves or imports by another mechanism before promising performance or cleanup behaviour.

For download-client authentication troubleshooting, see [[Hermes Media Stack - Container Connectivity Troubleshooting]].

## Verified Compose paths and ports (2026-10-18)

```yaml
sabnzbd-media:
  image: lscr.io/linuxserver/sabnzbd:latest
  container_name: sabnzbd-media
  environment:
    - PUID=1000
    - PGID=100
    - TZ=Europe/London
  ports:
    - "8081:8080"
  volumes:
    - /var/lib/media-stack/sabnzbd/config:/config
    - /var/lib/media-stack/sabnzbd/incomplete:/incomplete-downloads
    - /mnt/ds3/media/downloads:/data/downloads
  restart: unless-stopped
  networks:
    - media
```

**Host port:** 8081. **Container port:** 8080. The Compose definition proves these mounts are exposed to SABnzbd, but does **not** by itself prove that the application has selected `/incomplete-downloads` and `/data/downloads` in its **Folders** settings. Confirm the configured paths and categories before claiming the entire pipeline has been tested.

Sonarr and Radarr each bind `/mnt/ds3/media:/data`; thus the same DS3 download directory is visible to them as `/data/downloads` if configured accordingly. This layout supports consistent path references across the acquisition and import services.

## Verified application settings (2026-10-18)

The live `sabnzbd.ini` was read without exporting credentials. Its `[misc]` settings confirm `download_dir=/incomplete-downloads`, `complete_dir=/data/downloads`, `host=::`, `port=8080`, `direct_unpack=1`, and `host_whitelist=25cb1bec7b3d, sabnzbd-media`. The allowlist value is a recorded snapshot; the first entry may change when a container is recreated.

Together with the verified Compose binds, incomplete downloads use `/var/lib/media-stack/sabnzbd/incomplete` on the Pi and completed downloads use `/mnt/ds3/media/downloads` on DS3. Sonarr and Radarr both see completed downloads at `/data/downloads`.

**Category verification remains open:** the extraction returned an empty category object, but searched only for INI section names starting with `category:`. This does not establish that SABnzbd has no categories. The configured category names, any folder overrides, and end-to-end import/copy behaviour remain unverified.

## Operational evidence — recent successful imports (user confirmation)

The operator confirms that numerous new movies and television shows have been added successfully through the running stack recently. This is evidence of the download-to-library workflow working in normal operation, not merely passing an API connectivity test. A separate synthetic end-to-end download is therefore **not required to establish that the current pipeline works**. The exact SABnzbd category definitions and Seerr approval policy remain undocumented; their absence from this record does not imply a malfunction.

## Confirmed SABnzbd categories (operator UI, 2026-10-18)

The live SABnzbd Categories UI shows that relative paths are based on `/data/downloads`:

| Category | Relative folder | Resolved completed-download folder | Priority | Processing |
| --- | --- | --- | --- | --- |
| Default | Blank | `/data/downloads` | Normal | `+Delete` |
| `movies` | `movies` | `/data/downloads/movies` | Default | Default |
| `shows` | `shows` | `/data/downloads/shows` | Default | Default |

The screenshot shows no per-category script or indexer category/group restriction. These are **download staging destinations**, not the final Radarr/Sonarr library roots (`/data/movies` and `/data/shows`). The previous extraction script only searched for sections named `category:...` and therefore missed entries nested under SABnzbd's `[categories]` section. Its empty `categories` result was a parser limitation, not evidence that no categories existed.
