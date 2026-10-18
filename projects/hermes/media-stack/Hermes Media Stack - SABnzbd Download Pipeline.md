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

Hermes was configured around the principle that **incomplete downloads and repair work stay on the Raspberry Pi SSD**, while completed downloads are moved to NAS storage. This avoids performing temporary high-churn work directly on the network share.

The system also retained an existing, separate SABnzbd instance associated with DS2. The newer media-stack instance was named `sabnzbd-media` and exposed TCP 8080.

## Known implementation facts

- Pi SSD: staging location for incomplete work.
- Synology NAS: intended destination for completed work.
- Media library on DS3: `/mnt/ds3/media/movies` and `/mnt/ds3/media/shows`.
- DS2: existing SABnzbd storage context at `/mnt/ds2/main`.
- Sonarr and Radarr connect to the media SABnzbd instance through its API.
- During integration, the request from Sonarr initially received HTTP 403. Correcting the API key and SABnzbd hostname allowance resolved the issue.

The exact SABnzbd incomplete/completed path values, categories, Docker volume targets, and final import semantics were not preserved; they must be read from the deployed configuration.

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
