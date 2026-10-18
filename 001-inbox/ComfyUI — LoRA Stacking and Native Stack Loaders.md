---
type: reference
status: active
created: 2026-10-18
source:
  - https://github.com/Comfy-Org/ComfyUI/pull/16309
tags:
  - comfyui
  - lora
  - image-generation
  - model-conditioning
  - diffusion-models
---
## Overview

ComfyUI supports applying multiple LoRAs (Low-Rank Adaptations) to a diffusion model and, where supported, its text encoder.

**LoRA stacking is cumulative in its effect on model weights, but individual LoRA strengths are not automatically added together into a single strength value.**

Each LoRA contributes a scaled modification to the underlying model. These modifications may reinforce or interfere with one another depending on their relationship.

This article explains the underlying mathematics, practical implications, and implementation of the native ComfyUI LoRA stack loaders introduced through [ComfyUI PR #16309](https://github.com/Comfy-Org/ComfyUI/pull/16309).

## 1. How LoRA Stacking Works

### Mathematical model

For conventional additive LoRA application, the effective model weights can be represented as:

$$  
W_{\text{effective}} = W_{\text{base}} + \sum_{i=1}^{n} s_i \Delta W_i  
$$

Where:

- $W_{\text{base}}$ represents the original checkpoint weights.
    
- $\Delta W_i$ represents the weight modification introduced by LoRA $i$.
    
- $s_i$ represents the configured strength of LoRA $i$.
    
- $n$ represents the number of applied LoRAs.
    

Each LoRA contributes independently to the resulting weight modifications.

### Example

Consider three LoRAs applied to an Illustrious checkpoint:

|LoRA|Strength|Purpose|
|---|---|---|
|Gothic Horror|0.7|Atmosphere and aesthetics|
|Character Detail|0.6|Character appearance|
|Dramatic Lighting|0.5|Lighting characteristics|

The effective modifications are:

$$  
W_{\text{effective}} =  
W_{\text{base}}

- 0.7\Delta W_{\text{gothic}}
    
- 0.6\Delta W_{\text{character}}
    
- 0.5\Delta W_{\text{lighting}}  
    $$
    

**The strengths do not become a single combined strength of 1.8.**

Each strength controls its corresponding LoRA contribution.

Consequently, there is no general requirement to divide the available strength between all stacked LoRAs.

For example, three LoRAs do not need to be limited to strengths of 0.33 simply because three are active.

## 2. Cumulative Effects and LoRA Interference

Although individual strengths remain independent, their effects can interact.

### Reinforcing effects

When two LoRAs introduce similar modifications to overlapping model weights, their contributions may reinforce one another.

For example:

- LoRA A strengthens dramatic shadows.
    
- LoRA B also strengthens dramatic shadows.
    

Applying both may produce more pronounced shadows than either LoRA alone.

### Conflicting effects

LoRAs can also introduce modifications that oppose one another.

For example:

- LoRA A encourages high-contrast illumination.
    
- LoRA B encourages soft, flat illumination.
    

Their contributions may partially cancel, compete, or produce an unexpected visual result.

### Nonlinear generation behaviour

The underlying weight modifications are approximately additive under conventional LoRA application, but the resulting image is produced by a nonlinear neural network.

Therefore, visual effects cannot be predicted simply by adding strength values.

Two LoRAs at 0.7 may produce a more exaggerated result than one LoRA at 1.0, particularly when they influence similar concepts.

### Practical implications

- Tune each LoRA according to its contribution.
    
- Watch for overlapping concepts and unwanted exaggeration.
    
- Do not assume that reducing all strengths proportionally will produce the best result.
    
- Compare results using identical generation settings and seeds where possible.
    
- Introduce LoRAs individually before evaluating their combined effects.
    

## 3. Does LoRA Application Order Matter?

For conventional additive LoRAs applied to the same underlying model, the order of application should not materially change the resulting weight modifications.

Mathematically:

W + s_B\Delta W_B + s_A\Delta W_A  
$$

Therefore:

```
Base Model
  └── LoRA A (0.7)
       └── LoRA B (0.6)
```

should be effectively equivalent to:

```
Base Model
  └── LoRA B (0.6)
       └── LoRA A (0.7)
```

### Exceptions and caveats

Order independence assumes conventional additive LoRA behaviour.

Differences may arise from:

- Nonstandard adapter implementations.
    
- Model transformations between LoRA applications.
    
- Implementation-specific patching or merging.
    
- Floating-point rounding and numerical precision.
    
- Metadata handling or other non-weight state.
    

The mathematical equivalence should not be interpreted as a guarantee that every possible loader implementation produces identical results.

## 4. Native ComfyUI LoRA Stack Loaders

### Implementation reference

**Upstream PR:** [Comfy-Org/ComfyUI#16309 — Native LoRA Stack Loader Nodes](https://github.com/Comfy-Org/ComfyUI/pull/16309)

**Merged:** 9 October 2026

**Source file:** `comfy_extras/nodes_lora_stack.py`

The implementation introduces two native nodes:

|Node|Function|
|---|---|
|`Load LoRA (Model)`|Applies a stack of LoRAs to the diffusion model|
|`Load LoRA (Text Encoder)`|Applies a stack of LoRAs to the CLIP text encoder|

Both nodes support multiple LoRA entries within a single node.

### Supported configuration

Each entry contains:

|Property|Description|
|---|---|
|`lora_name`|LoRA file selection|
|`strength`|Individual LoRA strength|
|`enabled`|Enable or disable the entry|

The implementation defines:

- Minimum of 1 row.
    
- Maximum of 20 rows.
    
- Default strength of 1.0.
    
- Strength range of -100.0 to 100.0.
    
- Strength adjustment step of 0.01.
    
- Enabled by default.
    

A disabled entry retains its filename and configured strength.

### Execution behaviour

The native loader processes LoRAs sequentially.

The following excerpt is from the model loader implementation in PR #16309:

```
for row in loras:
    lora_name = row.get("lora_name")
    strength = row["strength"]

    if not lora_name or not row["enabled"] or strength == 0:
        continue

    lora, metadata = _load_lora_file(lora_name)

    model, _ = comfy.sd.load_lora_for_models(
        model,
        None,
        lora,
        strength,
        0,
        lora_metadata=metadata
    )
```

The important operation is:

```
model, _ = comfy.sd.load_lora_for_models(...)
```

The resulting model is passed into the next iteration.

Each enabled LoRA therefore contributes to the model already modified by preceding LoRAs.

**The native stack loader uses ComfyUI's existing LoRA application mechanism rather than introducing a new strength-combination algorithm.**

### Disabled and zero-strength entries

The loader skips an entry when:

- No LoRA filename is specified.
    
- The entry is disabled.
    
- Its strength is zero.
    

Skipped entries are not loaded or applied during execution.

### Negative strengths

Negative strengths are supported and passed to the existing LoRA application mechanism.

A negative strength reverses the sign of the LoRA's weight contribution. This does not guarantee a predictable visual opposite of the learned concept.

## 5. Model and Text Encoder Strengths

The native implementation separates diffusion-model and text-encoder LoRA application into different nodes.

### Model loader

The model loader calls:

```
model, _ = comfy.sd.load_lora_for_models(
    model,
    None,
    lora,
    strength,
    0,
    lora_metadata=metadata
)
```

The model receives the configured strength while the text-encoder strength is zero.

### Text encoder loader

The text encoder loader calls:

```
_, clip = comfy.sd.load_lora_for_models(
    None,
    clip,
    lora,
    0,
    strength,
    lora_metadata=metadata
)
```

The text encoder receives the configured strength while the model strength is zero.

### Practical significance

This separation allows independent control over the LoRA contributions to:

- The diffusion model, affecting image generation.
    
- The text encoder, affecting text conditioning where supported.
    

Not every LoRA contains applicable modifications for both components.

For a workflow previously using a loader with separate model and CLIP strengths, equivalent settings require configuring the corresponding native loaders appropriately.

## 6. Comparison with rgthree LoRA Loader Stacked

The existing workflow uses `rgthree LoRA Loader Stacked`.

Both the rgthree loader and the new native loader are intended to support multiple LoRAs without requiring a separate conventional loader node for every file.

|Capability|Native ComfyUI loader|
|---|---|
|Multiple LoRAs|Supported|
|Cumulative modifications|Supported|
|Individual strengths|Supported|
|Per-entry enable/disable|Supported|
|Negative strengths|Supported|
|Zero-strength skipping|Supported|
|Maximum entries|20|
|Model and CLIP control|Separate nodes|

The native implementation explicitly delegates LoRA application to `comfy.sd.load_lora_for_models()`.

This establishes that its fundamental application mechanism is consistent with ComfyUI's conventional LoRA loaders.

However, exact output equivalence with rgthree depends on the installed rgthree implementation and how model and text-encoder strengths are mapped.

**A direct comparison between the two loaders has not been performed in this discussion.**

## 7. Migration and Validation

The existing rgthree workflow does not require modification solely because the native loaders are available.

A controlled comparison is preferable before migration.

### Proposed test

1. Duplicate the existing workflow to preserve the known-good configuration.
    
2. Retain the original checkpoint, prompts, seed, sampler, scheduler, CFG, steps and resolution.
    
3. Replace the rgthree stacked loader with the native model loader and, where necessary, the native text-encoder loader.
    
4. Configure identical LoRA files and corresponding strengths.
    
5. Generate an image with the original workflow.
    
6. Generate an image with the native loader workflow.
    
7. Compare results for visual differences and execution errors.
    

### Expected result

For conventional LoRAs applied with equivalent strengths and conditioning, the results should be effectively equivalent.

Exact pixel-level equality is not guaranteed without controlling the execution environment and confirming equivalent loader behaviour.

### Validation status

|Item|Status|
|---|---|
|Native loader PR merged|Verified|
|Sequential application in source code|Verified|
|Per-entry strength and enable switch|Verified|
|Separate model and text encoder nodes|Verified|
|Upstream unit tests reported passing|Verified in PR|
|Full-model image-generation equivalence|Not established|
|rgthree-to-native migration test|Not performed|
|Availability in locally installed ComfyUI Core|Not established|

The PR reports 97 passing tests covering the loader and DynamicGroup functionality, as well as additional local CPU-level validation.

Its review notes explicitly distinguish those tests from full-model image generation and release integration testing.

## 8. Implementation Caveats

### Metadata handling

PR #16309 identifies a limitation involving metadata retention when multiple LoRAs are composed.

The issue is tracked in [ComfyUI PR #16834 — Preserve IC-LoRA metadata when composing multiple LoRAs](https://github.com/Comfy-Org/ComfyUI/pull/16834).

The native stack implementation does not introduce a stack-specific metadata merging policy.

This concern also applies to conventional chained loaders using the same underlying helper.

### Version availability

The PR was merged into ComfyUI Core on 9 October 2026.

This does not independently establish that the nodes are available in every ComfyUI Desktop installation.

The installed Core version and corresponding frontend support must be checked before assuming availability.

The discussion did not establish the exact locally installed Core version after the PR was merged.

## 9. Conclusions

1. **LoRA modifications are cumulative.** Each LoRA contributes a scaled modification to the effective model weights.
    
2. **Strengths are not a shared budget.** Multiple LoRAs do not need to divide a total strength of 1.0.
    
3. **Overlapping concepts can reinforce or interfere.** Visual results are nonlinear and cannot be predicted by summing strengths.
    
4. **Order is generally immaterial for conventional additive LoRAs.** Implementation-specific exceptions remain possible.
    
5. **The native ComfyUI stack loaders use the existing LoRA application mechanism.** They do not introduce a fundamentally different method of combining strengths.
    
6. **Model and text-encoder application are independently configurable.** The native implementation provides separate nodes for these paths.
    
7. **Migration from rgthree is optional.** Existing working configurations can be retained, with a controlled comparison used to establish equivalence before replacement.
    

## References

- [ComfyUI PR #16309 — Native LoRA Stack Loader Nodes](https://github.com/Comfy-Org/ComfyUI/pull/16309)
    
- [ComfyUI Frontend PR #17441 — DynamicGroup Widget Inputs](https://github.com/Comfy-Org/ComfyUI_frontend/pull/17441)
    
- [ComfyUI Frontend PR #20305 — Native LoRA Stack Asset Widgets](https://github.com/Comfy-Org/ComfyUI_frontend/pull/20305)
    
- [ComfyUI Frontend PR #20307 — Native LoRA Stack Integration Tests](https://github.com/Comfy-Org/ComfyUI_frontend/pull/20307)
    
- [ComfyUI PR #16834 — IC-LoRA Metadata Preservation](https://github.com/Comfy-Org/ComfyUI/pull/16834)