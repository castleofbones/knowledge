---

type: how-to  
status: active  
created: 2026-10-18  
source: ""  
tags:

- ai
    
- sillytavern
    
- docker
    
- raspberry-pi
    
- gluetun
    
- ollama
    
- patching
    
- maintenance
    

---
## Purpose

This runbook documents the deployment and maintenance of SillyTavern on the Raspberry Pi 5 (`hermes`), including its Docker networking, persistent storage and local compatibility patches.

The primary objectives are:

- Preserve configuration and user data across container recreation.
    
- Keep SillyTavern accessible through its Gluetun network arrangement.
    
- Preserve the Ollama compatibility changes without manually editing every new container.
    
- Make upgrades reversible and testable.
    
- Distinguish deployment failures from model and prompt problems.
    

For request construction and response handling, see [[SillyTavern Request and Response Pipeline]].

## 1. Deployment Architecture

The established deployment consists of:

|Component|Configuration|
|---|---|
|Host|Raspberry Pi 5, Debian 13|
|Container management|Docker and Portainer CE|
|SillyTavern image|`ghcr.io/sillytavern/sillytavern:latest`|
|VPN container|Gluetun|
|Network mode|`service:gluetun`|
|SillyTavern port|`8101`|
|Persistent storage|Synology NAS via `/mnt/ds3`|
|Inference engine|Ollama on Windows 11|
|Ollama endpoint|`http://10.10.0.83:11434`|
|Local model|Gemma 4 abliterated 12B QAT|

The relevant network arrangement is:

```
Browser
   │
   ▼
Raspberry Pi 5
   │
   ├── Gluetun network namespace
   │       │
   │       └── SillyTavern container
   │
   └── NAS-mounted persistent storage

SillyTavern
   │
   └── LAN connection to Windows Ollama
```

Because SillyTavern shares Gluetun's network namespace, network troubleshooting must account for Gluetun's routing and firewall configuration.

The VPN is used for SillyTavern; the wider media stack is not intended to share that VPN arrangement.

## 2. Persistent Storage

SillyTavern's persistent files are stored under:

```
/mnt/ds3/sillytavern/
```

The established layout includes:

```
/mnt/ds3/sillytavern/
├── config/
├── data/
├── plugins/
├── extensions/
└── patches/
    └── start-with-patches.sh
```

The directories are bind-mounted into the container.

This separates persistent state and custom startup logic from the replaceable container image.

### Why this matters

A Docker container's writable filesystem is not a reliable location for changes that must survive recreation.

Editing application files inside a running container can solve an immediate problem, but those edits are lost when the container is replaced.

The persistent patch directory provides a repeatable way to reapply necessary changes.

> [!warning]  
> Persistent storage does not automatically mean the deployment is backed up. Verify NAS availability and backup arrangements separately.

## 3. Startup Wrapper

The deployment uses a custom startup script:

```
/mnt/ds3/sillytavern/patches/start-with-patches.sh
```

Mounted inside the container as:

```
/patches/start-with-patches.sh
```

The container starts through `tini`, which invokes the wrapper.

The established entrypoint arrangement is:

```
entrypoint:
  - tini
  - --
  - /patches/start-with-patches.sh
```

The wrapper is responsible for applying local modifications before launching SillyTavern through its normal startup process.

### Important design principle

**Patch the application at startup, then hand control back to its normal entrypoint.**

Avoid replacing the application's startup logic unnecessarily.

The wrapper should perform only the compatibility work required by the deployment.

## 4. The Ollama Compatibility Patch

Our local Gemma 4 configuration required changes to SillyTavern's handling of reasoning-related parameters.

The relevant application files were:

```
/home/node/app/public/scripts/openai.js
/home/node/app/src/endpoints/backends/chat-completions.js
```

The changes supported forwarding the intended `reasoning_effort` configuration, including the value `none`, through the appropriate request path.

### Why two files were involved

The frontend and backend perform different responsibilities.

|File|Relevant responsibility|
|---|---|
|`public/scripts/openai.js`|Frontend request configuration and parameter handling|
|`src/endpoints/backends/chat-completions.js`|Backend request handling and forwarding|

A parameter appearing in the frontend configuration does not prove that it reaches Ollama.

Both sides of the request path must be checked.

### Patch scope

The patch should remain narrowly focused on the required compatibility behaviour.

Avoid unrelated modifications to:

- Prompt construction
    
- Generation presets
    
- Response formatting
    
- Streaming behaviour
    
- Authentication
    
- General application configuration
    

This makes regressions easier to diagnose and reduces maintenance work after updates.

## 5. Patch Idempotency

The startup patch must be safe to run repeatedly.

An **idempotent patch** produces the intended result whether it runs once or multiple times.

For example, the wrapper should not insert the same JavaScript modification every time the container starts.

A robust patch process should:

1. Check whether the expected source code is present.
    
2. Detect whether the patch is already applied.
    
3. Apply the patch only when necessary.
    
4. Verify that the expected modification exists.
    
5. Stop with a clear error if the source is incompatible.
    

### Avoid blind text replacement

A patch that simply searches for a string and replaces it can fail after an upstream update.

The source may have been reformatted, moved or rewritten.

Worse, an overly broad replacement may match the wrong location.

Prefer exact, validated patch operations with explicit success and failure conditions.

## 6. Startup Dependency Incident

We encountered a failure where the startup patch process repeatedly attempted to run because `python3` was unavailable in the container.

The application image did not contain the runtime dependency assumed by the patch script.

The issue was resolved by modifying the patch approach.

### Lesson learned

Do not assume that a container image contains general-purpose development tools.

A minimal application image may omit:

- Python
    
- Git
    
- Patch utilities
    
- Build tools
    
- Diagnostic utilities
    

Before introducing a startup dependency, verify that it exists in the actual image.

For simple source modifications, prefer an approach compatible with the tools already available, or explicitly manage the dependency.

Avoid adding unnecessary software to the application container merely to support a small patch.

## 7. Pre-Update Checklist

Before updating SillyTavern:

- Confirm the current container is healthy.
    
- Record the running image identifier.
    
- Record the SillyTavern version or commit where available.
    
- Preserve the current container configuration.
    
- Verify NAS mounts are available.
    
- Confirm the patch script is accessible.
    
- Review the patch's expected source patterns.
    
- Record a known-good Ollama request and response.
    

### Inspect the running image

```
docker inspect sillytavern \
  --format '{{.Image}}'
```

The container name must be adjusted to match the actual deployment.

### Inspect mounts

```
docker inspect sillytavern \
  --format '{{json .Mounts}}'
```

### Inspect startup configuration

```
docker inspect sillytavern \
  --format '{{json .Config.Entrypoint}}'
```

These checks help establish the current deployment before making changes.

## 8. Updating SillyTavern

The deployment uses Portainer for container management.

The general update sequence is:

1. Review the available SillyTavern update.
    
2. Confirm the existing deployment is healthy.
    
3. Record the current image and configuration.
    
4. Pull the replacement image.
    
5. Recreate the SillyTavern container with its existing mounts, networking and entrypoint.
    
6. Allow the startup wrapper to apply the compatibility patch.
    
7. Inspect startup logs.
    
8. Validate the application and Ollama connection.
    

### Important Docker behaviour

Pulling a newer image does not update an already-running container.

The container must be recreated to use the replacement image.

Similarly, restarting an existing container does not replace its image.

### Avoid uncontrolled updates

Using the `latest` image tag is convenient, but it does not provide a stable version identifier.

For reproducibility, record the image digest or immutable image ID associated with each known-good deployment.

A versioned image tag or digest is preferable where supported.

## 9. Post-Update Validation

A successful container start is only the beginning of validation.

### Stage 1 — Container health

Check that the container remains running:

```
docker ps
```

Inspect recent logs:

```
docker logs --tail 100 sillytavern
```

### Stage 2 — Application accessibility

Confirm that the SillyTavern web interface is reachable through the established Gluetun port arrangement.

### Stage 3 — Persistent data

Verify that existing:

- Characters
    
- Chats
    
- Presets
    
- Extensions
    
- Configuration
    

remain available.

### Stage 4 — Patch verification

Inspect the patched application files and confirm that the intended reasoning-related changes are present.

Do not rely solely on the wrapper reporting success.

### Stage 5 — Ollama connectivity

From an environment with the appropriate network access:

```
curl http://10.10.0.83:11434/api/tags
```

This confirms basic connectivity, not complete application functionality.

### Stage 6 — Generation test

Use a known-good SillyTavern conversation and preset.

Verify that:

- Generation succeeds.
    
- The model returns visible content.
    
- Streaming behaves as expected.
    
- Paragraph formatting is preserved.
    
- Reasoning configuration is handled correctly.
    
- The response is not unexpectedly truncated.
    

Where possible, compare the actual outbound request with a known-good capture.

## 10. Failure Diagnosis

|Symptom|Likely investigation area|
|---|---|
|Container exits immediately|Entrypoint and startup wrapper|
|Repeated startup loop|Patch script or missing dependency|
|Patch no longer applies|Upstream source changes|
|Web interface unavailable|Gluetun networking and port mapping|
|Existing data missing|Bind mounts and NAS availability|
|Ollama connection refused|Listener, LAN routing and firewall|
|Reasoning parameter missing|Frontend/backend patch compatibility|
|Empty model response|Request/response pipeline|
|Formatting regression|SillyTavern formatting configuration|
|Behaviour changes after update|Upstream changes, presets and patches|

These are starting points, not definitive diagnoses.

Investigate the first failing stage before changing unrelated settings.

## 11. Recovery and Rollback

If an update breaks the deployment, the preferred recovery is to return to the last known-good image and configuration.

### Rollback principles

- Preserve persistent data.
    
- Restore the previous image where available.
    
- Recreate the container using the recorded configuration.
    
- Reapply the compatible patch.
    
- Validate startup and generation.
    

### Critical limitation

An image rollback does not necessarily reverse changes to persistent configuration or data formats.

If an application update performs a data migration, an older image may not be compatible with the updated persistent data.

Review upstream migration requirements before assuming that container rollback is sufficient.

### Preserve known-good artefacts

Useful records include:

- Image digest or immutable image ID
    
- Container configuration
    
- Startup wrapper version
    
- Patch source and expected patterns
    
- SillyTavern version
    
- Successful validation results
    

These provide a reproducible recovery baseline.

## 12. Patch Retirement

A local patch should not become permanent merely because it works.

After each relevant upstream update, determine whether the original compatibility issue has been resolved.

The preferred sequence is:

1. Review upstream changes.
    
2. Test whether the required behaviour works without the local patch.
    
3. Compare the resulting outbound request.
    
4. Remove the patch only after successful verification.
    
5. Retain the previous patch version for rollback.
    

The ideal outcome is to eliminate the local modification once upstream SillyTavern provides the required behaviour.

This reduces future maintenance and compatibility risk.

## 13. Maintenance Principles

### Keep application state outside the container

Persistent data belongs on mounted storage, not in the container's writable layer.

### Keep patches minimal

Every custom modification introduces an additional maintenance obligation.

### Fail clearly

A patch that silently fails is more dangerous than one that stops with an explicit error.

### Validate behaviour, not just startup

A healthy container can still generate malformed requests.

### Record versions

Avoid relying on `latest` as the only record of a deployment.

### Separate concerns

Docker and Gluetun problems should be diagnosed independently from prompt construction and model inference.

## Related Notes

- [[SillyTavern Request and Response Pipeline]]
    
- [[SillyTavern Prompt Construction and Presets]]
    
- [[Quantisation, VRAM and Context]]
    
- [[LLM Generation Parameters — A Practical Guide]]
    
- [[Local Coding Agents]]
    

## Key Takeaway

**The SillyTavern container should be replaceable without losing application state or manually repeating compatibility changes.**

Persistent bind mounts, a minimal startup patch, recorded image versions and a repeatable validation procedure make that possible.

The most important maintenance rule is to treat every update as a controlled change: preserve the known-good state, apply the update, verify the patch and test actual model generation before declaring success.