---
type: how-to
status: draft
created: 2026-10-18
source: ""
tags:
  - synology-nas
  - nfs
  - docker
  - linux-storage
---

# Configuring Synology NFS Storage for Hermes Media Services

## Implemented topology

Hermes uses Synology NAS exports mounted on the Raspberry Pi host and then bind-mounted into Docker containers. The following host mount paths were discussed previously. The exact NFS export source addresses carried over from the initial draft were **not independently checked** against an original host configuration:

| Host path | NFS export | Role |
| --- | --- | --- |
| `/mnt/ds3/media` | Verify from live mount | Films and TV library |
| `/mnt/ds3/sillytavern` | Verify from live mount | Separate application data |
| `/mnt/ds2/main` | Verify from live mount | DS2 storage, including existing SABnzbd use |

The DS3 library used `/mnt/ds3/media/movies` and `/mnt/ds3/media/shows`. Both NAS and Pi were on the wired LAN.

## Verify before starting containers

```bash
findmnt -T /mnt/ds3/media -o TARGET,SOURCE,FSTYPE
findmnt -T /mnt/ds2/main -o TARGET,SOURCE,FSTYPE
ls -ld /mnt/ds3/media/movies /mnt/ds3/media/shows
```

Verify that `FSTYPE` is `nfs` or `nfs4` and that `SOURCE` matches the live mount unit or `/etc/fstab` entry—not the Pi's local filesystem. The earlier draft's numeric NAS IP addresses should not be copied without verification. For automounted paths, access may trigger the mount; a directory existing on disk alone is not evidence that NFS is attached.

To inspect relevant systemd units:

```bash
systemctl list-units --type=mount --type=automount
systemctl --failed
journalctl -b --no-pager | grep -i nfs
```

The exact `/etc/fstab` options and Synology export permissions were not preserved in the accessible history. Inspect them on the live system rather than copying generic mount options as if verified.

## Permissions and bind mounts

Several application configuration directories under `/var/lib/media-stack/` used UID 1000 and GID 100. NAS export identity mapping and Docker container users must be checked separately. A successful directory listing does not prove a service can write or rename completed downloads.

## Failure mode established after a power outage

Docker containers started or retained while NFS was unavailable could remain attached to the **underlying Pi SSD directory** even after the host NFS mount recovered. This is a mount-namespace issue, not simply a missing network share. A healthy host `findmnt` result is insufficient.

Inspect the actual running container mounts, and do not automatically restart containers with stale attachments if they may have written data to the underlay. The `media-recovery` utility was intentionally designed to report such states rather than repair them unattended.

## Verification checklist

- Host export and mount source match.
- Expected film/show folders are visible.
- Required container paths point at the intended NAS filesystem.
- Application identity can read/write only the directories it requires.
- No unexpected files have accumulated on the Pi SSD underlay.
- Recovery behaviour is documented for NFS loss.

**Evidence status:** The host paths and failure mode are supported by earlier discussion; exact NAS export source strings and a full Synology GUI setup are not independently verified. A complete Synology GUI export-configuration walkthrough is not documented by the available implementation record.

## Capture authoritative mount sources

```bash
grep -E 'nfs|/mnt/ds[23]' /etc/fstab
findmnt -t nfs,nfs4 -o TARGET,SOURCE,FSTYPE,OPTIONS
systemctl list-units --all --type=automount --type=mount
```

If the expected export is automounted, accessing its path may trigger the mount; distinguish the `.automount` unit from the NFS `.mount` unit.

## Verify the running container's filesystem view

```bash
docker inspect jellyfin --format '{{json .Mounts}}'
pid=$(docker inspect -f '{{.State.Pid}}' jellyfin)
test "$pid" -gt 0 && sudo findmnt -N "$pid" -t nfs,nfs4 -o TARGET,SOURCE,FSTYPE
```

Cross-check namespace paths with the container's bind destinations. A healthy host mount does not guarantee that a container created during an NFS outage sees that mount. Investigate possible local-underlay writes before any forced restart.

See [[Hermes Media Stack - Container Connectivity Troubleshooting]] and [[Hermes Media Stack - Operational Validation Runbook]].
