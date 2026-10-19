---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - comfyui
  - krea2
  - performance
  - benchmarking
  - vram
  - gpu
purpose: knowledge
---
## Overview

Investigation conducted on 14 October 2026 into ComfyUI Desktop's VRAM management, model caching, and generation performance on an NVIDIA RTX 5070 Ti.

**Outcome:** Retain `NORMAL_VRAM`. The available evidence does not justify switching to `HIGH_VRAM`. Warm generation is dominated by diffusion sampling rather than demonstrable model-loading overhead.

## Environment

|Component|Configuration|
|---|---|
|Operating system|Windows 11|
|Application|ComfyUI Desktop|
|ComfyUI Core|v0.40.0 (last reported)|
|GPU|NVIDIA RTX 5070 Ti, 16 GB|
|GPU memory reported by PyTorch|15.89 GiB|
|System RAM|32 GB|
|Python|3.13.12|
|PyTorch|2.12.1+cu130|
|VRAM mode|`NORMAL_VRAM`|

ComfyUI Desktop uses its managed Python environment. Startup arguments did not explicitly request `--highvram`, `--lowvram`, or `--gpu-only`.

The separate memory-manager query returned:

```
VRAM mode: VRAMState.NORMAL_VRAM
Total VRAM: 15.89 GiB
```

This establishes the default selected in a fresh Python process. It is consistent with the running application's reported `NORMAL_VRAM` state.

## Benchmark configuration

- Diffusion model: Krea 2 Turbo FP8 scaled
    
- Sampling steps: 8
    
- Base image-generation workflow
    
- No additional detailer or upscaling stages enabled for the benchmark
    
- Normal ComfyUI execution caching
    
- Subsequent executions triggered by changing the seed
    

## Benchmark results

|Job|Duration|Interpretation|
|---|---|---|
|#0|50.66 s|Cold execution|
|#1|0.04 s|Cached node outputs|
|#2|0.02 s|Cached node outputs|
|#3|27.24 s|Warm generation|
|#4|27.10 s|Warm generation|

**Average warm execution: 27.17 seconds.**

The first execution was approximately 23.5 seconds slower than the warm average.

The two warm executions differed by only 0.14 seconds, providing a repeatable baseline.

### Execution caching versus model caching

ComfyUI can reuse outputs from unchanged nodes.

Consequently, the near-instantaneous executions in Jobs #1 and #2 were not evidence of exceptionally fast diffusion sampling.

Changing the seed invalidated the sampler's cached output and caused sampling to execute again.

These are distinct optimisation mechanisms:

- **Node-output caching:** avoids recomputing unchanged workflow nodes.
    
- **Model caching:** avoids repeatedly loading or transferring model weights.
    

Benchmarks must force the relevant computation to execute while preserving other parameters.

## GPU telemetry

The following measurements were collected using `nvidia-smi` at approximately one-second intervals.

|Metric|Observed|
|---|---|
|Peak GPU memory usage|15,012 MiB|
|Approximate peak usage|14.66 GiB|
|Available physical memory|16,303 MiB|
|Remaining headroom at peak|~1,291 MiB|
|Maximum GPU utilisation|100%|
|Peak power draw|~252 W|
|Peak temperature|64°C|
|VRAM usage after generation|~10,136 MiB|

During the main sampling period, GPU utilisation remained at 100% for an extended interval, with power consumption around 250 W.

This supports the conclusion that the main sampling workload was keeping the GPU busy.

The retained VRAM allocation after generation is consistent with caching or retained model data. The telemetry alone does not identify which models or allocations remained resident.

## Execution log analysis

### Dynamic model loading

The log reported:

```
Model Krea2TEModel_ prepared for dynamic VRAM loading.
8463MB Staged.
```

And:

```
Model Krea2 prepared for dynamic VRAM loading.
12530MB Staged.
```

These values represent staged model data, not simultaneous measured VRAM allocations.

They nevertheless demonstrate why dynamic loading is important when using these models on a 16 GB GPU.

### Sampling performance

Warm executions reported approximately 23 seconds for eight sampling steps.

After model initialisation, individual sampling steps settled at approximately 3.3 seconds each.

The log also reported an initialisation phase of approximately 3–3.5 seconds on warm executions.

The sampler accounted for approximately 85% of total warm execution time.

### Other processing

The initial execution included text-encoder preparation, token generation, diffusion-model preparation, and VAE processing.

Warm executions avoided much of the initial overhead.

The logs did not provide sufficient timing detail to attribute the entire cold-versus-warm difference to any single operation.

## Findings

### Confirmed observations

1. ComfyUI selected `NORMAL_VRAM` without an explicit VRAM-mode override.
    
2. The first execution took approximately 50.7 seconds.
    
3. Warm executions took approximately 27.2 seconds.
    
4. Unchanged workflows could complete in milliseconds using cached outputs.
    
5. Warm sampling took approximately 23 seconds.
    
6. GPU utilisation reached and sustained 100% during sampling.
    
7. Peak VRAM usage approached the GPU's physical memory limit.
    
8. ComfyUI prepared both the text encoder and diffusion model for dynamic VRAM loading.
    

### Interpretation

The workload appears predominantly compute-bound during warm sampling.

There is no demonstrated substantial model-transfer bottleneck that would justify forcing more aggressive GPU residency.

The available evidence therefore does not support expecting a meaningful performance improvement from `HIGH_VRAM`.

This does not prove `HIGH_VRAM` could never help another workflow, especially one involving different model transitions.

## Configuration decision

**Retain** `**NORMAL_VRAM**`**.**

Do not introduce `--highvram` or `--gpu-only` based on the current measurements.

Reasons:

- Warm sampling already achieves sustained high GPU utilisation.
    
- VRAM headroom is relatively limited.
    
- Dynamic model loading is functioning.
    
- Warm execution times are consistent.
    
- No significant avoidable offloading overhead has been demonstrated.
    

The current configuration provides a known-good baseline for future comparisons.

## Next investigation: Attention performance

The next candidate for performance analysis is the attention implementation used by the diffusion model.

The logs explicitly reported:

```
Using pytorch attention in VAE
```

This confirms the VAE's attention selection but does **not** establish the diffusion model's attention backend.

Future investigation should establish:

1. The actual diffusion attention implementation selected at runtime.
    
2. Which alternative implementations are supported by the installed ComfyUI Core and model.
    
3. Whether compatible alternatives provide a measurable performance benefit.
    
4. Whether any improvement introduces additional VRAM pressure or stability issues.
    

Do not install Triton, SageAttention, FlashAttention, or other optional dependencies merely because they are available. Confirm compatibility with the installed Windows, Python, PyTorch, CUDA and ComfyUI versions first.

### Benchmark methodology

Preserve the established workflow, resolution, model precision, sampler, steps and other generation parameters.

Change one performance-related variable at a time.

Use warm executions and ensure the sampler actually runs.

Compare execution time, GPU utilisation, VRAM usage, image output and stability.

## Baseline summary

|Parameter|Established baseline|
|---|---|
|VRAM management|`NORMAL_VRAM`|
|Model|Krea 2 Turbo FP8 scaled|
|Steps|8|
|Warm execution|27.17 s average|
|Sampling|~23 s|
|Peak VRAM|14.66 GiB|
|GPU utilisation|Sustained 100%|
|Decision|No VRAM-mode changes|
|Next area|Diffusion attention backend|

## Conclusion

ComfyUI's existing VRAM management is behaving sensibly for the tested workload.

The cold-to-warm improvement is substantial, but warm generation is dominated by diffusion computation.

There is currently no evidence-based reason to modify the VRAM startup configuration.

**Preserve this baseline and investigate attention performance separately.**