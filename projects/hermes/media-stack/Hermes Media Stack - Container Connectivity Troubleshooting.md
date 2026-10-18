---
type: troubleshooting
status: active
created: 2026-10-18
source: ""
tags:
  - docker-networking
  - sabnzbd
  - sonarr
  - radarr
---

# Troubleshooting Hermes Media Stack Container Connectivity

## Case study: SABnzbd returned HTTP 403

During Sonarr/Radarr integration, a test connection to `sabnzbd-media` returned **403 Forbidden**. An HTTP request from inside Sonarr also received a 403. The request therefore reached an HTTP endpoint; the failure was not simply DNS failure or connection refusal.

The recorded resolution was to correct the SABnzbd API key and hostname allowance. Subsequent integration tests passed.

## Diagnostic sequence

1. Identify the actual container names and Docker network membership.
2. Check whether the hostname resolves **inside the requesting container**.
3. Distinguish connection refusal, timeout, DNS failure and HTTP 403.
4. Inspect the receiving application's host allowlist and API authentication.
5. Re-run the Sonarr/Radarr application-level test.

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}'
docker inspect sonarr
docker inspect sabnzbd-media
docker logs --tail 100 sonarr
docker logs --tail 100 sabnzbd-media
```

A diagnostic HTTP 403 is valuable evidence: transport succeeded far enough to receive an application response. Do not expose API keys in commands, screenshots or notes.

## Separate issue: artwork not appearing

Sonarr and Radarr libraries were populated but poster images initially did not appear. Requests to `skyhook.sonarr.tv` and `api.radarr.video` returned HTTP 200 in connectivity checks. This shows the tested endpoints were reachable, but does **not** identify the final artwork failure cause. Avoid documenting an unverified fix.

## Separate issue: NFS bind mounts

A healthy Docker container can still reference an underlying local directory if NFS was absent when its mount namespace was established. This is not a Docker DNS or API authentication fault. Inspect both host and container mounts before restarting anything after a storage outage.

## Expected result

After the recorded SABnzbd key and hostname corrections, Sonarr/Radarr tests passed. Future incidents should be classified by layer—DNS, TCP, HTTP authentication, application configuration or storage—before attempting fixes.

## Inspect mount namespaces for the separate storage failure

```bash
docker inspect jellyfin --format '{{json .Mounts}}'
pid=$(docker inspect -f '{{.State.Pid}}' jellyfin)
test "$pid" -gt 0 && sudo findmnt -N "$pid" -o TARGET,SOURCE,FSTYPE
```

These commands inspect metadata and mounts without restarting services. Compare the expected bind destination to the host's current NFS filesystem. They do not replace checking for files stranded on local underlay paths before attempting recovery. See [[Hermes Media Stack - Synology NFS Storage]].

## Quick error classification

| Observation | Likely layer | Next check |
| --- | --- | --- |
| DNS lookup fails | Name resolution/network | Same-network container DNS and aliases |
| Connection refused | Listener/port/network | Service status and bound port |
| HTTP 403 | HTTP application auth/allowlist | API key and permitted hostname |
| API passes, media absent | Library path/storage | Bind sources, container namespace, import paths |

These are diagnostic categories, not exclusive diagnoses.
