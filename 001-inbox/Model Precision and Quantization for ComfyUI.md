---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- model-quantization
    
- floating-point-precision
    
- nvidia-blackwell
    
- gpu-memory
    

purpose: knowledge
---
## Overview

AI model releases frequently offer several representations of the same model, including BF16, FP8, INT8 and NVFP4.

These formats determine how model weights are represented numerically, influencing:

- Model storage requirements
    
- GPU VRAM consumption
    
- Inference performance
    
- Numerical accuracy and potentially output quality
    
- Hardware and software compatibility
    

Choosing a lower-precision format can substantially reduce memory requirements without necessarily producing a noticeable reduction in image quality.

The optimal format depends on the model, its quantization implementation, available hardware and inference software.

## Hardware Context

The local image-generation environment at the time of this discussion:

|Component|Configuration|
|---|---|
|Operating system|Windows 11|
|Application|ComfyUI Desktop|
|ComfyUI Core|v0.40.0 (last confirmed)|
|GPU|NVIDIA RTX 5070 Ti|
|Architecture|NVIDIA Blackwell|
|VRAM|16 GB|
|System RAM|32 GB DDR5|
|PyTorch|2.12.1+cu130 (last confirmed)|

These are historical configuration values, not guarantees of the currently installed versions.

## Numerical Formats

### BF16 — Brain Floating Point

BF16 is a 16-bit floating-point representation.

Its bit allocation is:

|Component|Bits|
|---|---|
|Sign|1|
|Exponent|8|
|Fraction|7|

BF16 preserves the exponent range of FP32 while sacrificing significand precision.

**Characteristics:**

- Commonly used in model training and inference.
    
- Provides a useful high-precision reference for comparing quantized variants.
    
- Requires approximately two bytes per parameter.
    
- Can exceed available VRAM for large models.
    

BF16 is not mathematically lossless relative to FP32, but it is generally a useful reference when evaluating lower-precision model releases.

### FP8 — 8-bit Floating Point

FP8 reduces weight storage to approximately one byte per parameter.

Two common representations are:

|Format|Sign|Exponent|Fraction|
|---|---|---|---|
|E4M3|1|4|3|
|E5M2|1|5|2|

E4M3 offers greater significand precision, while E5M2 provides a wider numerical range.

**Characteristics:**

- Approximately half the raw weight storage of BF16.
    
- Frequently retains image quality close to higher-precision variants.
    
- Can reduce CPU offloading when model weights fit within GPU memory.
    
- Performance depends on implementation and hardware support.
    

FP8 is a strong starting point for inference on GPUs with constrained VRAM.

### INT8 — 8-bit Integer

INT8 represents quantized values as integers, typically in the range -128 to 127.

A simplified reconstruction formula is:

```
original_value ≈ scale × (quantized_value - zero_point)
```

For symmetric quantization, the zero point may be zero.

Example with scale = 0.01:

|Original|Stored|Reconstructed|
|---|---|---|
|0.253|25|0.25|
|-0.718|-72|-0.72|
|1.127|113|1.13|

Real implementations may use per-tensor, per-channel or per-block scaling.

**Characteristics:**

- Approximately one byte per parameter before metadata.
    
- Requires quantization and reconstruction rules.
    
- Can offer efficient inference with supported kernels.
    
- Is not automatically faster or more accurate than FP8.
    

### NVFP4 — NVIDIA 4-bit Floating Point

NVFP4 is a low-precision representation associated with NVIDIA Blackwell hardware.

It uses 4-bit floating-point values together with scaling metadata to approximate higher-precision model weights.

**Characteristics:**

- Approximately half the raw weight storage of FP8.
    
- Designed for efficient execution on supported NVIDIA hardware.
    
- Can enable larger models to fit within limited VRAM.
    
- May introduce greater quantization error than FP8.
    
- Requires compatible model packaging and inference kernels.
    

The RTX 5070 Ti supports the relevant Blackwell hardware capabilities, but successful NVFP4 execution also depends on software support.

NVFP4 should not automatically be assumed faster than FP8.

## Model Size Comparison

Consider a hypothetical model with 12 billion parameters.

|Format|Raw weight storage|
|---|---|
|BF16|24 GB|
|FP8|12 GB|
|INT8|12 GB|
|NVFP4|6 GB|

These values are calculated from parameter count and bits per parameter.

Actual model files and runtime allocations can differ because of:

- Quantization scales and metadata
    
- Mixed-precision layers
    
- Model architecture
    
- Tensor alignment
    
- Runtime buffers
    
- Activation memory
    
- Other loaded models
    

### Practical Example: 25 GB BF16 vs 12 GB FP8

A model release offered two relevant variants:

- BF16: approximately 25 GB
    
- FP8: approximately 12 GB
    

For the RTX 5070 Ti with 16 GB VRAM, FP8 is the preferred starting point.

The BF16 weights alone exceed available GPU memory and would require some form of offloading or alternative execution strategy.

The FP8 model has a much better chance of fitting predominantly or entirely in GPU memory.

However, a 12 GB file does not guarantee a 12 GB runtime VRAM footprint.

## Model Precision vs Image Resolution

Two separate memory considerations must be distinguished.

**Model precision** primarily determines the memory required to represent model weights.

**Image resolution** affects the size of intermediate activations, latent representations and other processing buffers.

Generating smaller images can reduce runtime memory consumption but does not proportionally reduce the model's weight storage.

Likewise, FP8 is not inherently less appropriate for small images or BF16 inherently more appropriate for large images.

Output resolution and numerical precision are independent choices, although both contribute to overall resource requirements.

## ComfyUI Workflow Considerations

The existing image-generation workflows include processing stages such as:

- Base image generation
    
- Eye detailing
    
- Manual inpainting
    
- Optional upscaling
    
- Preview and output routing
    

These stages can introduce additional memory requirements.

Reducing model-weight memory consumption can leave more VRAM available for intermediate tensors and additional models.

However, memory requirements depend on which components remain resident in VRAM simultaneously.

### Preferred Format Selection

For the RTX 5070 Ti:

1. **FP8:** Preferred initial choice when a suitable variant exists.
    
2. **NVFP4:** Consider when FP8 requires excessive offloading, encounters memory limitations or when native NVFP4 execution offers a demonstrated performance advantage.
    
3. **BF16:** Use when the model fits comfortably or when a higher-precision comparison is needed.
    
4. **INT8:** Evaluate when the particular model and inference implementation provide a suitable optimized variant.
    

This is a practical selection strategy, not a universal ranking of numerical formats.

## Relationship to GGUF

GGUF models often use quantization labels such as:

- `Q8_0`
    
- `Q6_K`
    
- `Q4_K_M`
    
- `Q3_K_M`
    

These are specific quantization schemes, rather than interchangeable names for FP8, INT8 or NVFP4.

For example, `Q4_K_M` uses block-based quantization with mixed quantization choices.

An NVFP4 model and a GGUF Q4 model may have broadly comparable weight-storage requirements, but their numerical representations and execution paths differ.

Their relative speed and output quality cannot be inferred from nominal bit width alone.

## Understanding Model Release Labels

### Version Prefixes

A release labelled `V5 FP8` may indicate version 5 of a particular model distributed in FP8 format.

The `V5` component is a model-author version designation, not a numerical precision format.

Its exact meaning must be established from the release documentation.

### QUANTS

`QUANTS` is shorthand for quantizations.

It does not identify a specific numerical representation.

In the model release examined during this discussion, the author used `QUANTS` to group the non-BF16 variants.

This was a release-organization convention rather than a separate model architecture or precision format.

Other authors may organize quantized releases differently.

## Recommended Comparison Procedure

To compare quantized variants:

1. Load the same model version in different precision formats.
    
2. Use the same positive and negative prompts.
    
3. Preserve seed, sampler, steps, CFG, resolution and other generation settings.
    
4. Measure peak VRAM usage.
    
5. Measure generation time after accounting for model-loading and warm-up effects.
    
6. Compare visual fidelity, consistency and artifacts.
    

Identical seeds do not guarantee pixel-identical outputs across precision formats or execution kernels.

The objective is to establish whether quality differences are meaningful for the intended workflow.

### Expected Outcome

FP8 may provide image quality close enough to BF16 that the reduction in memory consumption is substantially more valuable than any visible difference.

NVFP4 may provide additional memory and performance benefits, but this requires testing with the actual model and runtime.

## Conclusions

For a 16 GB RTX 5070 Ti running ComfyUI Desktop:

- FP8 is the sensible default for a model offered as approximately 12 GB FP8 or 25 GB BF16.
    
- BF16 is useful as a reference but is not practical for keeping that entire model in VRAM.
    
- NVFP4 is a promising alternative when memory becomes limiting.
    
- Model precision and output image resolution affect different aspects of memory consumption.
    
- Quantization quality and execution efficiency depend on the specific implementation.
    
- `QUANTS` is a packaging label, not a numerical format.
    

The best choice is the lowest-cost representation that delivers acceptable quality and stable performance in the actual workflow.