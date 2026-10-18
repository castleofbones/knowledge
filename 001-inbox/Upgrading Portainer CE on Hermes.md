---

type: how-to  
status: active  
created: 2026-10-18  
source: ""  
tags:

- raspberry-pi
    
- docker
    
- portainer
    
- container-lifecycle
    
- shell-scripting
    

---
## Overview

Portainer Community Edition (CE) runs as a Docker container on **Hermes**, a Raspberry Pi 5 running Debian 13 (Trixie).

Upgrading Portainer requires replacing its existing container with one created from the updated image. Restarting the existing container is insufficient because Docker containers retain the image ID used at creation.

The established upgrade procedure preserves Portainer's persistent data volume, Docker socket access, HTTPS port mapping and restart policy.

Two approaches are documented:

1. **Manual upgrade** — successfully executed and verified.
    
2. **Update-aware shell script** — proposed automation that skips unnecessary container recreation.
    

## Existing deployment

The following configuration was confirmed using `docker inspect` before the manual upgrade.

|Property|Value|
|---|---|
|Host|Hermes (Raspberry Pi 5)|
|Container|`portainer`|
|Image|`portainer/portainer-ce:latest`|
|HTTPS port|`9443:9443`|
|Data volume|`portainer_data:/data`|
|Docker socket|`/var/run/docker.sock:/var/run/docker.sock`|
|Restart policy|`always`|
|Deployment|Docker CLI|

The `portainer_data` Docker-managed named volume persists independently of the Portainer container.

It contains Portainer's configuration and application state. Removing the container with `docker rm` does not remove this volume.

**Important:** Never remove `portainer_data` as part of a routine upgrade.

## Why pulling and restarting is insufficient

Docker containers are created from specific image IDs.

Running:

```
docker pull portainer/portainer-ce:latest
docker restart portainer
```

does not upgrade Portainer.

The pull updates the local `latest` image reference, but the existing container continues using its original image.

The required operation is:

```
Pull updated image
        |
        v
Stop existing container
        |
        v
Remove existing container
        |
        v
Create replacement from updated image
        |
        v
Attach existing persistent volume
        |
        v
Verify replacement
```

### Kubernetes analogy

The conceptual equivalent is deploying a replacement workload rather than restarting an existing pod.

|Docker operation|Kubernetes concept|
|---|---|
|`docker restart`|Restarting the existing workload without changing its image|
|Pulling an image|Making an image available to the runtime|
|Recreating a container|Creating a replacement workload|
|Docker named volume|Persistent storage, conceptually similar to PV/PVC|
|Docker Compose reconciliation|Declarative workload reconciliation|

These are conceptual comparisons rather than exact mappings between Docker and Kubernetes abstractions.

### Why Docker Compose behaves differently

With a Compose-managed service:

```
docker compose pull
docker compose up -d
```

Compose can detect an image change and recreate the affected container automatically.

A plain Docker restart does not perform this reconciliation.

## Manual upgrade procedure

**Status: Tested successfully on Hermes.**

### 1. Inspect existing configuration

```
docker inspect portainer --format \
'Image={{.Config.Image}}
Ports={{json .HostConfig.PortBindings}}
Mounts={{json .Mounts}}
Restart={{.HostConfig.RestartPolicy.Name}}'
```

Confirm the existing image, port mappings, mounts and restart policy.

Check the persistent volume:

```
docker volume ls | grep portainer
```

Expected:

```
local     portainer_data
```

### 2. Stop and remove Portainer

```
docker stop portainer
docker rm portainer
```

Verify the volume remains:

```
docker volume ls | grep portainer
```

### 3. Pull the updated image

```
docker pull portainer/portainer-ce:latest
```

During the successful upgrade, Docker returned:

```
Status: Downloaded newer image for portainer/portainer-ce:latest
```

The image digest recorded for that pull was:

```
sha256:4d616db18cfeb5dd41a69c0958bc825c84483ea9cde1106eb82a5d26f3bd8b0e
```

This digest documents the image downloaded during the original upgrade, not the current latest release.

### 4. Recreate Portainer

```
docker run -d \
  --name portainer \
  --restart=always \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

### 5. Verify the deployment

```
docker ps --filter name=portainer
```

Then access Portainer over HTTPS on port `9443`.

The upgrade was confirmed successful when the Portainer UI displayed the existing configuration and managed resources as expected.

**Verified result:** The updated Portainer container started successfully, reused the persistent volume and retained the existing configuration.

## Automated upgrade script

**Status: Proposed; not yet tested on Hermes.**

The automation improves the manual procedure by comparing the image ID of the existing container against the image ID obtained after pulling `latest`.

If the IDs match, no container replacement is required.

### Proposed location

```
~/scripts/update-portainer.sh
```

### Script

```
#!/usr/bin/env bash
set -euo pipefail

IMAGE="portainer/portainer-ce:latest"
CONTAINER="portainer"
VOLUME="portainer_data"

echo "Checking Portainer for updates..."

# Capture the image ID used by the existing container.
CURRENT_IMAGE_ID=$(docker inspect \
    --format '{{.Image}}' \
    "$CONTAINER")

echo "Current image: ${CURRENT_IMAGE_ID:7:12}"

# Pull the latest available image.
docker pull "$IMAGE"

# Resolve the image ID behind the updated tag.
LATEST_IMAGE_ID=$(docker image inspect \
    --format '{{.Id}}' \
    "$IMAGE")

echo "Latest image:  ${LATEST_IMAGE_ID:7:12}"
echo

# Avoid unnecessary recreation.
if [[ "$CURRENT_IMAGE_ID" == "$LATEST_IMAGE_ID" ]]; then
    echo "Portainer is already up to date."
    exit 0
fi

echo "New Portainer image available."
echo "Recreating container..."

docker stop "$CONTAINER"
docker rm "$CONTAINER"

docker run -d \
    --name "$CONTAINER" \
    --restart=always \
    -p 9443:9443 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$VOLUME:/data" \
    "$IMAGE"

echo
echo "Waiting for Portainer to start..."
sleep 5

if docker ps \
    --filter "name=^/${CONTAINER}$" \
    --filter "status=running" \
    --format '{{.Names}}' | grep -qx "$CONTAINER"; then

    echo "Portainer container successfully recreated."

    docker ps \
        --filter "name=^/${CONTAINER}$" \
        --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
else
    echo "Portainer failed to start."
    echo "Last 50 log lines:"
    docker logs --tail 50 "$CONTAINER" || true
    exit 1
fi
```

### Installation

Create the script in the intended location and make it executable:

```
mkdir -p ~/scripts
chmod +x ~/scripts/update-portainer.sh
```

Execute:

```
~/scripts/update-portainer.sh
```

### Expected outcomes

**No update available**

```
Checking Portainer for updates...
Current image: abc123456789
Latest image:  abc123456789

Portainer is already up to date.
```

No container recreation occurs.

**Update available**

The script pulls the updated image, replaces the container and verifies that the replacement is running.

**Failure**

The script exits with a nonzero status if a command fails or the replacement container is not running at verification time.

The current implementation does not automatically restore the previous container.

## Script limitations

The proposed script is deliberately simple, but has important limitations:

- **No automatic rollback:** If the replacement fails, the previous container is not restored.
    
- **Running is not ready:** `docker ps` confirms the container is running, not that Portainer is responding successfully over HTTPS.
    
- **Fixed configuration:** The script assumes the existing port, mounts and restart policy remain unchanged.
    
- **No concurrency protection:** Simultaneous executions are not prevented.
    
- **Mutable image tag:** `latest` follows the upstream image and does not pin a specific Portainer release.
    
- **No automatic downgrade safety:** Portainer may migrate persistent application data during an upgrade. Reusing that data with an older image may not be supported.
    

These limitations do not invalidate the tested manual procedure, but should be considered before unattended execution.

## Future improvements

### Automatic rollback

The image ID of the existing container is already captured before the pull:

```
CURRENT_IMAGE_ID=$(docker inspect \
    --format '{{.Image}}' \
    "$CONTAINER")
```

This could be used to recreate the previous container if the replacement fails.

However, an image rollback alone does not guarantee compatibility with application data that may have been migrated.

Rollback should therefore be designed and tested separately rather than assumed safe.

### Application readiness check

Replace the fixed five-second delay and container-running check with a bounded readiness test against Portainer's HTTPS endpoint or a supported health mechanism.

This would distinguish between a running Docker process and a usable Portainer application.

### Preserve deployment configuration

A more general upgrade script could inspect and preserve the actual existing container configuration rather than hardcoding it.

Alternatively, migrating the Portainer deployment to Docker Compose would make the configuration declarative and reduce custom upgrade logic.

## Operational considerations

Portainer manages other Docker services on Hermes, including the media stack.

Upgrading the Portainer management container does not itself require stopping the workloads it manages.

Avoid broad Docker cleanup operations such as:

```
docker system prune
```

as part of this upgrade procedure.

Unused images can be inspected separately:

```
docker image ls -f dangling=true
```

Image cleanup should be deliberate, particularly if retaining previous images for possible recovery.

## Conclusions

The manual Portainer CE upgrade on Hermes was successful.

The key technical conclusions are:

1. Pulling an updated image does not update an existing Docker container.
    
2. A replacement container must be created from the updated image.
    
3. Docker named volumes persist independently of container recreation.
    
4. Comparing the existing container image ID with the newly pulled image ID avoids unnecessary redeployments.
    
5. Docker Compose can automate container reconciliation, while a Docker CLI deployment requires explicit recreation.
    
6. The proposed update-aware script remains untested and does not yet provide safe automatic rollback.
    

The recommended next step is to test the update-aware script on Hermes, initially with no update available, before considering unattended execution or additional recovery logic.