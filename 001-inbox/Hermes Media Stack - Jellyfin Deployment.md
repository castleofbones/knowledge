---
type: how-to
status: active
created: 2026-10-18
source: ""
tags:
  - jellyfin
  - raspberry-pi
  - docker
  - media-playback
purpose: knowledge
---

# Jellyfin on Hermes — Verified Deployment and Playback

## Environment and observed result

Jellyfin was deployed on the Hermes Raspberry Pi 5 as a Docker container managed by Portainer CE. The image was recorded as `latest` at the time of the build, with TCP **8096** and UDP **7359** exposed. The container was healthy and playback was verified on a television.

Jellyfin reads films and television content from the Synology DS3 library, whose host directories are `/mnt/ds3/media/movies` and `/mnt/ds3/media/shows`.

## Revalidate the deployment

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
docker inspect jellyfin
docker logs --tail 100 jellyfin
findmnt -T /mnt/ds3/media -o TARGET,SOURCE,FSTYPE
```

The supplied Compose definition now establishes the container-side library paths, user identity, persistent configuration directories and network. Use `docker inspect` to confirm the currently running container matches the declared service.

## Playback verification

1. Confirm that Jellyfin reports a healthy container and its UI is reachable.
2. Confirm film and series libraries resolve to the NAS-backed paths.
3. Play representative content on the TV client.
4. Review the Jellyfin logs during playback to identify direct play, remux or transcoding.

In the recorded successful test, the video stream was copied while audio was transcoded using `libfdk_aac`, delivered through HLS. This confirms that the playback pipeline worked for that particular item/client; it does **not** establish that hardware video transcoding was enabled.

## Storage failure caveat

After NFS interruption, a running container may retain a stale mount namespace even when the host's NFS mount is healthy. Verify the filesystem attached **inside Jellyfin** after an outage rather than relying only on the host's mount table.

## Evidence gaps

The exact Compose service and container paths are now documented below. Jellyfin's application-level library names and currently deployed image digest remain unverified. The supplied service defines no hardware acceleration device mapping.

## Deployment reconstruction worksheet

The **verified** baseline is the service on TCP 8096/UDP 7359 and successful playback. Before describing an exact redeployment, record the live values of image, network, configuration mount, cache mount, media-library bind destinations, user identity, and any device mappings:

```bash
docker inspect jellyfin --format 'Image={{.Config.Image}} User={{.Config.User}} Network={{.HostConfig.NetworkMode}}'
docker inspect jellyfin --format '{{json .Mounts}}'
docker inspect jellyfin --format '{{json .HostConfig.Devices}}'
docker inspect jellyfin --format '{{json .HostConfig.PortBindings}}'
```

Do not assume that the observed audio transcoding proves hardware acceleration. Video copy and AAC audio conversion establish only that the tested playback pipeline worked.

For NAS-dependent service validation see [[Hermes Media Stack - Synology NFS Storage]].

## Confirmed Compose service (2026-10-18)

```yaml
jellyfin:
  image: jellyfin/jellyfin:latest
  container_name: jellyfin
  user: "1000:100"
  ports:
    - "8096:8096"
    - "7359:7359/udp"
  volumes:
    - /var/lib/media-stack/jellyfin/config:/config
    - /var/lib/media-stack/jellyfin/cache:/cache
    - /mnt/ds3/media/movies:/media/movies:ro
    - /mnt/ds3/media/shows:/media/shows:ro
  restart: unless-stopped
  networks:
    - media
```

This is the actual service stanza supplied by the user. The media binds are **read-only**, the service runs as UID 1000/GID 100, and the host exposes 8096/tcp and 7359/udp. The `media` bridge network is declared by the parent Compose file. Hardware acceleration devices were **not** configured in this stanza. Playback observations still need to be interpreted independently of the YAML.
