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

Hermes uses Synology NAS exports mounted on the Raspberry Pi host and then bind-mounted into Docker containers. The user supplied both current `findmnt` output and `/etc/fstab` entries on 2026-10-18. The following three exports are now **confirmed** as mounted NFSv4 filesystems:

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

Verify that `FSTYPE` is `nfs` or `nfs4` and that `SOURCE` matches the live mount unit or `/etc/fstab` entry—not the Pi's local filesystem. These sources have now been cross-checked against the supplied live mount table and `/etc/fstab`. For automounted paths, access may trigger the mount; a directory existing on disk alone is not evidence that NFS is attached.

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

**Evidence status:** The three active NFS export source strings and mount targets are verified from the supplied `findmnt` and `/etc/fstab` output. Synology-side export permissions and GUI configuration remain undocumented. A complete Synology GUI export-configuration walkthrough is not documented by the available implementation record.

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

## Verified automount definitions

The relevant supplied `/etc/fstab` entries are:

```fstab
10.10.0.252:/volume1/media /mnt/ds3/media nfs4 defaults,_netdev,nofail,x-systemd.automount 0 0
10.10.0.252:/volume1/sillytavern /mnt/ds3/sillytavern nfs4 defaults,_netdev,nofail,x-systemd.automount 0 0
10.10.0.253:/volume1/main /mnt/ds2/main nfs4 defaults,_netdev,nofail,x-systemd.automount 0 0
```

The live mount output reports NFSv4.0 with `hard`, `timeo=600`, `retrans=2`, TCP and `sec=sys`. Other `/etc/fstab` entries cover DS3 `pishare`, DS2 USB exports and a CIFS USB share; these are outside the media project's critical path.

**Observed state requiring separate diagnosis:** `mnt-ds3-media.mount` was **active/mounted** while `mnt-ds3-media.automount` was **inactive/dead**. A separate `mnt-ds2-usb3.mount` was failed. Do not equate these states with a broken media share: the DS3 media NFS filesystem was mounted at capture time. Check whether the automount unit's state is expected after mount activation before changing it.
