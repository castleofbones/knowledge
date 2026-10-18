---
type: how-to
status: active
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

Hermes uses Synology NAS exports mounted on the Raspberry Pi host and then bind-mounted into Docker containers. The confirmed mounts from the later recovery configuration are:

| Host path | NFS export | Role |
| --- | --- | --- |
| `/mnt/ds3/media` | `10.10.0.252:/volume1/media` | Films and TV library |
| `/mnt/ds3/sillytavern` | `10.10.0.252:/volume1/sillytavern` | Separate application data |
| `/mnt/ds2/main` | `10.10.0.253:/volume1/main` | DS2 storage, including existing SABnzbd use |

The DS3 library used `/mnt/ds3/media/movies` and `/mnt/ds3/media/shows`. Both NAS and Pi were on the wired LAN.

## Verify before starting containers

```bash
findmnt -T /mnt/ds3/media -o TARGET,SOURCE,FSTYPE
findmnt -T /mnt/ds2/main -o TARGET,SOURCE,FSTYPE
ls -ld /mnt/ds3/media/movies /mnt/ds3/media/shows
```

Check that `SOURCE` is the expected NFS export, not the Pi's local filesystem. For automounted paths, access may trigger the mount; a directory existing on disk alone is not evidence that NFS is attached.

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

**Status:** The topology and failure behaviour are verified. A complete Synology GUI export-configuration walkthrough is not documented by the available implementation record.
