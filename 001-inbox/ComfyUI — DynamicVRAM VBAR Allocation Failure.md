---
type: troubleshooting
status: archived
created: 2026-10-18
source: ""
tags:
  - comfyui
  - comfyui-desktop
  - dynamicvram
  - nvidia-cuda
  - memory-management
  - windows-11
purpose: knowledge
---
## Overview

On 15 October 2026, ComfyUI Desktop encountered a reproducible memory allocation failure while executing a Krea 2 Turbo image-generation workflow.

The failure occurred during text encoder model loading, before image sampling, and originated in the `comfy-aimdo` DynamicVRAM memory-management implementation.

The exception was:

```
MemoryError: VBAR allocation failed
```

The available evidence identifies the failing operation but does not establish the underlying cause.

A recent NVIDIA driver update is a possible contributing factor, although no causal relationship has been demonstrated.

**Investigation status:** Unresolved.

## Environment

The following configuration was captured in the error report and subsequent diagnostics.

|Component|Version / Configuration|
|---|---|
|Operating system|Windows 11|
|Installation|ComfyUI Desktop|
|ComfyUI Core|0.40.0|
|Python|3.13.12|
|PyTorch|2.12.1+cu130|
|comfy-aimdo|0.5.6|
|comfy-kitchen|0.2.37|
|GPU|NVIDIA GeForce RTX 5070 Ti|
|GPU memory|16 GB|
|NVIDIA driver|617.42|
|CUDA driver version reported by `nvidia-smi`|13.4|
|GPU driver model|WDDM|
|DynamicVRAM|Enabled|

The NVIDIA driver version was captured after a driver update. The previously installed driver version was not recorded in this investigation.

### Relevant models

The execution logs identified:

**Diffusion model:**

```
krea2_turbo_fp8_scaled.safetensors
```

**Text encoder:**

```
qwen3vl-4b-abliterated_fp8_e4m3fn.safetensors
```

**VAE:**

```
qwen_image_vae.safetensors
```

The logs also show a SAM model being loaded:

```
sam_vit_b_01ec64.pth
```

The execution failure occurred while loading `Krea2TEModel_`.

## Symptoms

The workflow failed at:

```
Node ID: 77:61
Node Type: CLIPTextEncode
Exception Type: MemoryError
Exception Message: VBAR allocation failed
```

The underlying error was:

```
aimdo: src/model-vbar.c:242:
ERROR: Could not reseve Virtual Address space for VBAR
```

The word `reseve` is preserved from the original diagnostic output.

The failure was reproduced twice without a successful intervening execution being documented.

### Execution timeline

|Timestamp|Event|
|---|---|
|21:13:54|First workflow execution begins|
|21:13:56|`Krea2TEModel_` loading requested|
|21:14:37|VBAR allocation failure|
|21:14:38|Execution terminated after 43.60 seconds|
|21:20:02|Second workflow execution begins|
|21:20:02|`Krea2TEModel_` loading requested|
|21:20:43|VBAR allocation failure|
|21:20:43|Execution terminated after 41.69 seconds|
|21:22:49|GPU state inspected using `nvidia-smi`|

Both executions failed at the same internal allocation operation.

## Technical Analysis

### Failure location

The Python stack trace identifies the following call path:

```
CLIPTextEncode
  → encode_from_tokens_scheduled()
  → encode_from_tokens()
  → load_model()
  → load_models_gpu()
  → model_load()
  → model_use_more_vram()
  → partially_load()
  → load()
  → _vbar_get()
  → comfy_aimdo.model_vbar.ModelVBAR()
```

The relevant ComfyUI Core operation was:

```
vbar = comfy_aimdo.model_vbar.ModelVBAR(
    self.model_size() * 10,
    self.load_device.index
)
```

The failure originated in:

```
comfy_aimdo/model_vbar.py
```

At the native layer, the error was reported from:

```
src/model-vbar.c:242
```

The stack trace therefore identifies a failure to reserve virtual address space during model loading.

### DynamicVRAM

Startup logs confirmed:

```
DynamicVRAM support detected and enabled
comfy-aimdo version: 0.5.6
```

The VBAR allocation operation uses a size argument derived from ten times the model size.

The precise purpose and allocation behaviour of this multiplier have not been independently verified against the source code for the installed version.

However, reserving virtual address space is conceptually distinct from committing an equivalent amount of physical GPU memory.

Consequently, a VBAR reservation failure must not automatically be interpreted as conventional GPU VRAM exhaustion.

### GPU memory observations

After the failures, the following command was executed:

```
nvidia-smi
```

The output reported:

```
Driver Version: 617.42
CUDA Version: 13.4

GPU: NVIDIA GeForce RTX 5070 Ti
Driver Model: WDDM

Memory Usage: 554 MiB / 16303 MiB
GPU Utilization: 0%
Temperature: 39°C
Power: 9W / 300W
```

Approximately 15.4 GiB of reported GPU memory was unused at the time of inspection.

These observations demonstrate that the GPU was effectively idle **after** the failed executions.

They do not establish GPU memory availability at the exact moment the VBAR allocation was attempted.

They also do not establish the state of process virtual address reservations.

### Backend process state

The `nvidia-smi` process listing included:

```
Comfy Desktop.exe
```

No ComfyUI Python backend process appeared in that listing.

This is not sufficient to conclude that the backend had terminated, particularly under Windows WDDM, where process reporting has limitations.

The state of the Python backend remained unverified.

## Custom-Node Involvement

The stack trace passed through execution wrappers belonging to:

- `comfyui-lora-manager`
    
- `comfyui-cyberdelia-metadata`
    

Both extensions intercepted portions of the execution path before control reached the native model-loading implementation.

Neither extension was identified as the source of the exception.

There is insufficient evidence to attribute the failure to either extension.

No custom-node removal or replacement is justified by the current findings.

## Possible Causes

The following are hypotheses, not confirmed diagnoses.

### 1. DynamicVRAM virtual address reservation failure

**Evidence:**

- DynamicVRAM was enabled.
    
- `comfy-aimdo` initiated the failed reservation.
    
- Both executions failed at the same internal operation.
    

**Unknowns:**

- The requested reservation size in bytes.
    
- Available GPU virtual address space at failure time.
    
- Whether the allocation failed because of resource constraints, fragmentation, driver behaviour, or an implementation defect.
    

### 2. NVIDIA driver interaction or regression

An NVIDIA driver update occurred before or around the period in which the failure was investigated.

The installed driver was subsequently identified as:

```
617.42
```

The failure involves GPU virtual memory management, making the driver a plausible area of investigation.

However:

- The previous driver version is unknown.
    
- The exact update time relative to the last successful execution has not been established.
    
- No comparison against the previous driver has been performed.
    
- No matching upstream regression has been verified.
    

A driver regression must therefore remain an unconfirmed hypothesis.

### 3. Process-level memory or allocation state

The failure might be affected by the state of the running ComfyUI backend, including prior model loads or resource reservations.

The available evidence does not establish whether restarting the backend changes the outcome.

This hypothesis requires a controlled reproduction.

## Recommended Investigation

The following steps are proposed and have not been recorded as completed.

### Step 1 — Restart ComfyUI Desktop

1. Save the current workflow.
    
2. Fully exit ComfyUI Desktop.
    
3. Confirm the associated Python backend processes have terminated.
    
4. Relaunch ComfyUI Desktop.
    
5. Execute the same Krea 2 workflow without changing its configuration.
    

**Expected diagnostic outcomes:**

- **Execution succeeds:** Suggests that process lifetime or transient allocation state may contribute to the problem.
    
- **VBAR failure recurs:** Demonstrates that the failure survives a fresh backend process and warrants further investigation.
    

Neither outcome alone conclusively identifies the root cause.

### Step 2 — Capture memory state during execution

Run:

```
nvidia-smi -l 1
```

Observe GPU memory usage during model loading, particularly immediately before the failure.

Also inspect the ComfyUI Python process in Windows Task Manager, including its memory consumption.

**Limitation:** `nvidia-smi` reports GPU memory usage but does not directly establish the amount or layout of virtual address space available for VBAR reservations.

### Step 3 — Establish the driver update timeline

Determine:

- The previously installed NVIDIA driver version.
    
- Whether the last successful Krea 2 execution occurred before the update.
    
- Whether the first VBAR failure occurred immediately after the update.
    

This information would help assess whether the driver change is a meaningful regression candidate.

### Step 4 — Investigate version-specific implementation behaviour

If the failure persists, inspect the implementation and relevant upstream issues for:

- ComfyUI Core 0.40.0
    
- `comfy-aimdo` 0.5.6
    
- NVIDIA driver 617.42
    

Focus specifically on VBAR reservation failures and DynamicVRAM model-loading behaviour.

Any proposed configuration change or workaround must be verified against the installed versions before use.

## Changes Not Yet Justified

The available evidence does not currently justify:

- Reinstalling ComfyUI Desktop.
    
- Replacing the Krea 2 workflow.
    
- Removing custom nodes.
    
- Replacing checkpoints or text encoders.
    
- Disabling DynamicVRAM without first verifying the supported mechanism and implications.
    
- Rolling back the NVIDIA driver.
    

These interventions introduce additional variables without an established causal basis.

## Current Conclusions

### Confirmed

1. ComfyUI Core 0.40.0 encountered a reproducible `MemoryError` during Krea 2 text encoder loading.
    
2. Both failures occurred inside `comfy-aimdo` VBAR virtual address reservation.
    
3. DynamicVRAM was enabled, with `comfy-aimdo` 0.5.6 installed.
    
4. The failure occurred before sampling.
    
5. GPU memory usage was low when inspected after the failed executions.
    
6. NVIDIA driver 617.42 was installed at the time of the subsequent diagnostic inspection.
    

### Not established

1. Whether physical GPU memory pressure contributed to the failure.
    
2. Whether the Python backend remained running after the exception.
    
3. Whether the NVIDIA driver update caused a regression.
    
4. Whether the problem survives a clean ComfyUI Desktop restart.
    
5. Whether the issue is specific to Krea 2 or affects other model-loading paths.
    
6. Whether a ComfyUI or `comfy-aimdo` defect is responsible.
    

## Resolution

**Unresolved as of 18 October 2026.**

No verified fix has been established.

The next diagnostic action is to reproduce the failure after a complete ComfyUI Desktop restart, preserving the existing workflow and model configuration.

Once resolved, this article should be updated with the verified cause, corrective action, and successful validation results.

---

## Key Takeaway

A `VBAR allocation failed` exception in ComfyUI's DynamicVRAM model-loading path is not equivalent to a conventional CUDA out-of-memory error.

Virtual address reservation and physical VRAM consumption are distinct operations.

Troubleshooting should identify the failing allocation mechanism and establish the relevant runtime and driver state before changing model configurations, memory-management settings, or GPU drivers.