---

type: reference  
status: active  
created: 2026-10-18  
source:  
tags:

- comfyui
    
- krea-2
    
- image-generation
    
- workflow-architecture
    
- subgraphs
    
- lora
    
- sampling
    
- inpainting
    
- detailing
    
- reproducibility
    

---
> [!abstract] Overview  
> This article documents the modular Krea 2 image-generation workflow developed through practical experimentation in ComfyUI Desktop.
> 
> The workflow began as a conventional text-to-image pipeline and evolved into a reusable architecture supporting model configuration, prompt conditioning, deterministic sampling, automated eye refinement, manual masked inpainting, and conditional routing.
> 
> The central design principle is **separation of responsibilities**: each subgraph performs a meaningful operation, while the parent workflow coordinates how those operations interact.
> 
> This article records our tested configuration, explains the processing path, distinguishes architectural decisions from implementation details, and identifies areas for future development.

## 1. Environment and Scope

The workflow was developed using ComfyUI Desktop on Windows 11.

### Development environment

|Component|Recorded configuration|
|---|---|
|Operating system|Windows 11|
|Installation|ComfyUI Desktop|
|ComfyUI Core|v0.40.0|
|Python|3.13.12|
|PyTorch|2.12.1+cu130|
|GPU|NVIDIA RTX 5070 Ti|
|GPU VRAM|16 GB|
|System RAM|32 GB|
|Workflow family|Krea 2 Extended|

These values describe the environment recorded during development in October 2026.

They should not be treated as proof of currently installed versions.

ComfyUI Core, Desktop, Manager and custom-node packages can be updated independently.

### Scope

This workflow focuses on still-image generation and refinement.

It is not intended to document the separate Krea 2 video-generation experiments or the complete behaviour of every Krea-compatible checkpoint.

## 2. Architectural Objectives

The workflow evolved around several practical requirements.

### Reusability

Generation and refinement operations should be usable across different prompts and characters.

### Separation of concerns

Model loading, conditioning, sampling, detailing and routing should remain independent responsibilities.

### Reproducibility

Seeds and generation parameters should be controllable so that experiments can be compared.

### Optional refinement

The user should be able to select an unmodified base image, automated eye refinement or manual masked repair.

### Observability

Intermediate previews should remain available where they provide useful diagnostic information.

### Replaceability

A failing custom-node implementation should be replaceable without requiring the surrounding workflow to be redesigned.

### Simplicity at the top level

The parent workflow should communicate the overall processing architecture rather than expose every implementation detail.

## 3. High-Level Architecture

The workflow is organised into reusable components.

```
                    Model Loader
                         |
              +----------+----------+
              |          |          |
            MODEL       CLIP        VAE
              |          |          |
              +----------+----------+
                         |
                    Conditioning
                         |
                         v
                     Base Image
                         |
                         v
                  Generated IMAGE
                         |
              +----------+----------+
              |          |          |
              v          v          v
             Base       Eye       Masked
             Path     Detailer    Detailer
              |          |          |
              +----------+----------+
                         |
                         v
                 Routing Subgraph
                         |
                         v
                    Final Output
```

This is a logical overview.

The actual workflow contains additional connections, including conditioning, model resources, masks, sampling parameters and shared routing controls.

### Main components

|Component|Responsibility|
|---|---|
|Model Loader|Load and configure model resources|
|Conditioning|Convert prompts into model conditioning|
|Base Image|Generate and decode the initial image|
|Eye Detailer|Automatically refine facial detail, particularly the eyes|
|Masked Detailer|Perform manually controlled inpainting and compositing|
|Routing|Select the required processing path|
|Output|Preview or save the selected result|

The architecture allows the main generation process to remain stable while individual refinement operations are developed independently.

## 4. Model Loader

The Model Loader subgraph centralises the resources required for generation.

Our original Krea 2 workflow used:

|Resource|Recorded value|
|---|---|
|Diffusion checkpoint|`realismByStableYogi_v30INT4Extended.safetensors`|
|CLIP|`qwen3vl_4b_fp8_scaled.safetensors`|
|CLIP type|`krea2`|
|VAE|`qwen_image_vae.safetensors`|
|Model sampling node|`ModelSamplingAuraFlow`|
|Sampling shift|3|

The checkpoint was presented in the workflow as Muse by Stable Yogi — Extended.

### Resource responsibilities

**Diffusion model**

Performs the image-generation and refinement operations.

**CLIP / text encoder**

Produces representations of the supplied prompt text for conditioning.

**VAE**

Converts between image pixels and latent representations.

**Model sampling configuration**

Applies the sampling configuration required by the selected model workflow.

### Why centralise these resources?

The Base Image and detailing components need compatible generation resources.

Providing them from a shared Model Loader reduces the chance of accidentally configuring different branches with incompatible checkpoints, VAEs or conditioning systems.

### Model-specific configuration

The AuraFlow sampling shift of 3 was part of our tested configuration.

It should not be treated as a universal value for every model or checkpoint.

The model-loading architecture can be reused, but model-specific settings still need to be validated.

## 5. LoRA Integration

We investigated stacked LoRA loading as part of the Model Loader design.

The objective was to support specialised visual influences without repeatedly replacing the entire base checkpoint.

### Architectural placement

Conceptually:

```
             Base MODEL
                  |
                  v
              LoRA Loader <---- LoRA Configuration
                  |
                  v
           Configured MODEL
                  |
                  v
            Model Sampling
                  |
                  v
             Generation
```

Some LoRA loaders also process CLIP or text-encoder resources.

The exact connection path depends on the loader implementation and the model architecture.

### Why place LoRAs near model loading?

It keeps model configuration centralised.

This makes it easier to understand which adaptations are active and helps avoid inconsistent configurations between generation and refinement.

### Our experiments

We investigated `rgthree` stacked LoRA loading, including its MODEL and CLIP connections.

However, we did not establish that every LoRA or loader configuration was compatible with our Krea 2 checkpoint.

### Important distinction

A LoRA is not a generic patch that can be applied interchangeably across all model architectures.

Compatibility depends on factors such as:

- The architecture used during training
    
- The target layers
    
- The loader implementation
    
- The base checkpoint
    
- The intended strength settings
    

### Practical principle

Preserve the stable base model and introduce compatible LoRAs when a specific visual requirement justifies them.

Do not replace the entire checkpoint merely because one detail is difficult to obtain, unless the checkpoint itself is unsuitable for the intended result.

## 6. Conditioning

The Conditioning subgraph converts prompt text into the representations used by the sampler.

Our workflow used the Krea 2-compatible Qwen3-VL text-encoding configuration.

### Inputs

Conceptually:

- Positive prompt
    
- Negative prompt
    
- CLIP resource
    

### Outputs

- Positive conditioning
    
- Negative conditioning
    

These outputs can be shared between the base generator and refinement components.

### Why separate conditioning?

Prompt development is an independent activity from sampling.

Keeping conditioning separate allows us to experiment with:

- Danbooru-style tags
    
- Natural-language prompts
    
- Prompt enhancement
    
- Different character descriptions
    
- Clothing and material specifications
    
- Composition instructions
    

without changing the underlying sampling architecture.

### Prompting philosophy

Our preferred approach evolved into:

1. Start with Danbooru tags to establish the visual inventory.
    
2. Expand the tags into natural language.
    
3. Specify physical and spatial relationships.
    
4. Remove unnecessary ambiguity.
    
5. Generate and inspect.
    
6. Refine the prompt only where the problem is caused by interpretation.
    

See [[Krea 2 — Prompt Engineering]].

## 7. Prompt Enhancement

We also experimented with prompt enhancement using Qwen3-VL.

The purpose was to convert user prompts or tags into more detailed descriptions suitable for image generation.

### Intended behaviour

The enhancer should:

- Preserve the requested subject
    
- Preserve important visual attributes
    
- Clarify relationships
    
- Improve descriptive precision
    
- Avoid inventing unrelated details
    

### Observed behaviour

The Qwen3-VL model was successfully used for text prompt enhancement.

We also investigated image-to-prompt generation and found that enabling thinking in the relevant template improved output reliability.

In our tests, this also improved the stability of the Danbooru-related processing.

### Architectural lesson

Prompt enhancement belongs before conditioning.

The enhanced text should remain inspectable so that unintended changes can be identified.

### Future consideration

A useful comparison is to generate images from:

- The original prompt
    
- The Danbooru-derived prompt
    
- The enhanced natural-language prompt
    

while keeping the remaining generation parameters controlled.

This would help distinguish genuine improvements in specification from stylistic changes introduced by the enhancer.

## 8. Base Image Generation

The Base Image subgraph handles initial text-to-image generation.

Its core operations are:

1. Allocate the initial latent.
    
2. Run diffusion sampling.
    
3. Decode the latent into an image.
    

### Processing path

```
             Empty Latent Image
                      |
                      v
MODEL ------------> KSampler
                      ^
                      |
                 CONDITIONING
                      |
                      v
                 LATENT Output
                      |
                      v
                  VAE Decode
                      |
                      v
                     IMAGE
```

The KSampler receives the latent, model and conditioning inputs together.

### Recorded configuration

|Parameter|Value|
|---|---|
|Width|960|
|Height|1440|
|Steps|12|
|CFG|1.5|
|Sampler|Euler|
|Scheduler|Simple|
|Denoise|1.0|
|AuraFlow shift|3|

These settings formed our successful baseline for the Stable Yogi Extended checkpoint.

### Why 960 × 1440?

The dimensions provide a portrait-oriented image with sufficient detail for our character experiments.

The resolution is within the general 1024-pixel-class generation range we investigated.

It is not necessary for both dimensions to equal 1024.

### Why denoise 1.0?

The Base Image path begins with an empty latent rather than an encoded source image.

The sampler therefore performs a full text-to-image generation.

Denoise 1.0 was appropriate for that operation.

## 9. Noise, Seeds and Reproducibility

A seed controls the initial pseudorandom noise used by the sampling process.

We used fixed seeds during development to compare workflow changes.

One recorded seed from the original workflow was:

`1007100245080363`

### Why use a fixed seed?

It reduces one source of variation between generations.

This makes it easier to investigate the effect of changing:

- Prompt wording
    
- CFG
    
- Step count
    
- Model configuration
    
- LoRA settings
    
- Refinement parameters
    

### What a fixed seed does not guarantee

A fixed seed does not guarantee identical images when other parameters change.

It also does not guarantee identical results across different model versions, execution environments or implementations.

### Our experimental method

We generally preferred:

1. Preserve the known-good configuration.
    
2. Keep the seed fixed.
    
3. Change one relevant parameter.
    
4. Generate a new result.
    
5. Compare the output.
    
6. Retain or reject the change.
    

This helped us distinguish prompt effects from sampling and workflow effects.

See [[ComfyUI — Noise Seeds and Reproducibility]].

## 10. Why We Added Refinement

Our initial café portrait was visually convincing in several respects.

The lighting, clothing, hair, skin and overall composition were successful.

However, the eyes were not consistently anatomically plausible.

This created a useful distinction between two problems.

### Prompt interpretation

The model understands what the image should contain.

### Rendering quality

The model generates the requested feature correctly.

The prompt successfully specified green eyes.

The resulting eye anatomy still required improvement.

### Initial experiment

We attempted whole-image image-to-image refinement.

At low denoise settings, this produced undesirable crosshatched textures in areas such as hair, clothing and skin.

The process modified too much of the image without reliably addressing the localised defect.

### Architectural response

Rather than continuing to regenerate the whole image, we introduced two independent refinement strategies:

- Manual masked inpainting
    
- Automated eye detailing
    

Both operate downstream of the base image.

## 11. Masked Detailer

The Masked Detailer provides manually controlled image repair.

Its processing path includes:

- Mask input
    
- VAE encoding for inpainting
    
- Diffusion sampling
    
- VAE decoding
    
- Masked image compositing
    

### Conceptual processing

```
Original IMAGE ------+
                     |
MASK ----------------+--> VAE Encode for Inpainting
                     |                |
VAE -----------------+                v
                                   LATENT
                                      |
                                      v
                                   KSampler
                                      |
                                      v
                                  VAE Decode
                                      |
                                      v
                                Repaired IMAGE
                                      |
                                      v
                              ImageCompositeMasked
                                      ^
                                      |
                               Original IMAGE
                                      |
                                      v
                                 Final IMAGE
```

### Our experiment

We painted a mask over the defective eyes using the ComfyUI Mask Editor.

The mask covered the eye regions while avoiding unnecessary surrounding areas.

We used:

|Parameter|Value|
|---|---|
|Steps|12|
|CFG|1.5|
|Sampler|Euler|
|Scheduler|Simple|
|Denoise|1.0|
|`grow_mask_by`|6|

### Why denoise 1.0?

The mask restricted the intended repair region.

Within that region, we wanted substantial regeneration rather than minor adjustment.

### Compositing

The repaired image was combined with the original using `ImageCompositeMasked`.

This allowed the original image to be preserved outside the selected mask during the final composite operation.

### Reusability

The resulting subgraph accepts an arbitrary suitable image and mask.

Although developed for eye repair, it can be used for other localised corrections.

See [[ComfyUI — Masks and Compositing]].

## 12. Eye Detailer

The Eye Detailer provides automated refinement.

Its implementation uses:

- Ultralytics face detection
    
- SAM segmentation
    
- Impact Pack FaceDetailer
    
- Diffusion sampling
    

### Why call it Eye Detailer?

The original subgraph was named `Face Detailer`.

We renamed it `Eye Detailer` because its intended purpose in our workflow is to improve eye quality.

The underlying detector still identifies faces.

The name describes the component's responsibility rather than claiming that it performs eye-only segmentation.

### Detector

We used:

`bbox/face_yolov9c.pt`

Through `UltralyticsDetectorProvider`.

### Segmentation

We used:

`sam_vit_b_01ec64.pth`

Through `SAMLoader (Impact)`.

The SAM loader was connected to FaceDetailer's optional SAM input.

### Observed behaviour

Before SAM was connected, the mask preview showed a rectangular region.

After SAM was connected, the mask became face-shaped.

This demonstrated that segmentation was affecting the selected area.

### Crop-factor experiments

|Crop factor|Observed crop|
|---|---|
|3.0|Approximately 730 × 1065|
|2.0|Approximately 487 × 710|

We preferred 2.0 for the café portrait.

### Denoise experiments

At denoise 0.50, the eyes improved but the face changed more substantially.

At denoise 0.30, the refinement preserved more of the original character while improving the eyes.

### Selected configuration

|Parameter|Value|
|---|---|
|Detector|`face_yolov9c.pt`|
|SAM|`sam_vit_b_01ec64.pth`|
|Steps|12|
|CFG|1.5|
|Denoise|0.30|
|Crop factor|2.0|
|Guide size|512|
|Guide size basis|bbox|
|Maximum size|1024|

### Diagnostic output

We exposed FaceDetailer's `cropped_refined` output through the subgraph.

This allowed us to inspect the processed crop without opening the subgraph.

See [[ComfyUI — Automated Detailing]].

## 13. Routing Architecture

The workflow supports three refinement modes:

- Base image
    
- Eye Detailer
    
- Masked Detailer
    

We developed a shared-index inverse-switch/index-switch architecture to control the routing.

### Original implementation

The architecture was established using Impact Pack switches.

Both switches received the same integer selection value.

### Compatibility problems

Following ComfyUI updates, we encountered problems with the Impact switch implementations.

The issue affected switch execution even though the surrounding image-generation stages remained functional.

### Replacement implementation

We replaced the two switch nodes with equivalents from ComfyUI-Easy-Use:

- `Any Inversed Switch`
    
- `Any Index Switch`
    

The shared-index architecture remained unchanged.

### Current routing concept

```
                  Shared INT Index
                         |
               +---------+---------+
               |                   |
               v                   v
       Any Inversed Switch   Any Index Switch
               |                   ^
               |                   |
          Routed Paths ------------+
```

The actual routing subgraph also exposes the relevant image inputs and outputs.

### Why preserve the architecture?

The routing design was already established.

The problem was with the node implementation, not a demonstrated architectural flaw.

Replacing the nodes allowed us to retain the surrounding workflow.

### General lesson

Separate architectural responsibilities from the custom nodes that implement them.

This makes version-related compatibility problems easier to isolate.

See [[ComfyUI — Switches and Execution Flow]].

## 14. Subgraph Architecture

The workflow was progressively reorganised into reusable subgraphs.

### Model Loader

Encapsulates model-resource loading and configuration.

### Conditioning

Encapsulates prompt processing.

### Base Image

Encapsulates latent allocation, sampling and decoding.

### Eye Detailer

Encapsulates automated detection, segmentation and refinement.

### Masked Detailer

Encapsulates inpainting and compositing.

### Routing

Encapsulates conditional data flow.

### Why this matters

The top-level graph now communicates the major processing stages.

The internal implementation details remain available when required but do not dominate the parent workflow.

### Software-engineering principles

The architecture reflects several familiar concepts:

**Single responsibility**

Each component performs a meaningful operation.

**Dependency injection**

Shared model resources and conditioning are supplied to downstream components.

**Encapsulation**

Internal node complexity is hidden behind defined interfaces.

**Replaceability**

A custom-node implementation can be changed without necessarily redesigning the surrounding workflow.

**Observability**

Selected diagnostic outputs remain available for troubleshooting.

See [[ComfyUI — Subgraph Architecture]].

## 15. Execution Flow and Conditional Processing

Routing introduces an important distinction between selecting data and controlling execution.

A switch can select one result without necessarily preventing all other branches from executing.

Our current ComfyUI-Easy-Use implementation uses two complementary mechanisms.

### Inverse-switch execution blocking

The inspected upstream implementation returns `ExecutionBlocker(None)` for unselected outputs.

### Index-switch lazy evaluation

The inspected upstream implementation declares selectable inputs as lazy and requests the selected input.

### Why this matters

The combination provides conditional routing behaviour.

However, shared prerequisites and independently requested outputs can still influence which nodes execute.

### Practical verification

We should verify execution behaviour through actual workflow runs rather than assuming that visual routing alone guarantees computational savings.

The distinction is particularly important for expensive operations such as detailing and inpainting.

## 16. Reproducible Experimentation

Our development process relied heavily on controlled experiments.

### Baseline preservation

We retained known-good configurations while investigating new functionality.

### Fixed seeds

We used fixed seeds to reduce variation during comparisons.

### Incremental changes

We generally changed one relevant variable at a time.

### Diagnostic previews

We exposed intermediate images and crops to identify where unexpected behaviour originated.

### Evidence-led decisions

We selected settings based on generated results rather than assuming that conventional defaults would necessarily work best.

### Example: Eye Detailer denoise

We compared denoise 0.50 and 0.30 while preserving the broader configuration.

The lower value produced the better balance for our character.

### Example: Crop factor

We compared 3.0 and 2.0.

The smaller crop factor produced a tighter processing region, which we preferred.

### Example: Switch migration

We isolated the routing problem and replaced the switch implementation rather than rebuilding the entire workflow.

These experiments demonstrate the value of treating image-generation workflows as testable systems.

## 17. Separating Prompt Problems From Workflow Problems

One of the most important lessons was learning to identify which layer was responsible for an undesirable result.

### Prompt interpretation problem

The image contains the wrong object, clothing or spatial relationship.

Investigate the prompt.

### Rendering problem

The model understands the intended feature but renders it poorly.

Consider targeted detailing or inpainting.

### Routing problem

The correct image is generated internally but does not reach the intended output.

Investigate switches and dependencies.

### Model compatibility problem

A custom node, LoRA or resource cannot operate correctly with the selected architecture.

Investigate compatibility rather than changing unrelated prompts.

### Execution problem

A node fails, an output is missing, or the workflow cannot validate.

Inspect the actual failure and relevant logs.

### Diagnostic table

|Symptom|Primary investigation|
|---|---|
|Wrong clothing|Prompt specification|
|Incorrect courtyard perspective|Spatial prompting|
|Malformed eyes|Rendering and detailing|
|Unwanted changes across the image|Denoise and refinement scope|
|Correct intermediate image but missing final output|Routing|
|Workflow fails to open after update|Custom-node compatibility|
|LoRA has unexpected effect|Compatibility and strength|
|Different results between runs|Seed and configuration consistency|

The objective is to make the smallest justified change at the correct layer.

## 18. Performance Considerations

The workflow runs on an RTX 5070 Ti with 16 GB VRAM.

This is sufficient for the tested Krea 2 configuration, although actual memory requirements depend on the checkpoint format, image dimensions and active processing components.

### Model precision

We investigated model formats including:

- BF16
    
- FP8
    
- NVFP4
    
- INT4
    

Lower-precision formats can reduce memory requirements, but may involve trade-offs in quality, compatibility or performance.

The correct choice depends on the specific model implementation.

### Refinement cost

Detailing introduces additional processing beyond the base image generation.

Automated detailing may require:

- Detection
    
- Segmentation
    
- Crop preparation
    
- Additional sampling
    
- Compositing
    

Manual inpainting also requires additional sampling.

### Conditional execution

Routing can help avoid unnecessary processing when only one refinement path is required.

Actual execution behaviour should be verified with the installed switch implementations.

### Practical principle

Optimise based on observed bottlenecks.

Do not introduce architectural complexity merely because a theoretical optimisation might be possible.

## 19. Known Limitations and Future Development

The workflow provides a stable foundation for further experimentation, but several areas remain open.

### LoRA compatibility

Further testing is needed to establish which LoRAs and loaders work reliably with the selected Krea 2 checkpoint.

### Dedicated eye detection

The current Eye Detailer uses face detection and face-oriented segmentation.

A future implementation could investigate more targeted eye-region detection if the additional complexity is justified.

### Upscaling

The workflow can be extended with optional upscaling.

The appropriate approach depends on whether the objective is increasing resolution, adding detail or preserving the existing image.

### Prompt enhancement

Further controlled comparisons could help determine when prompt enhancement improves fidelity and when it introduces unwanted interpretation.

### Alternative checkpoints

The modular architecture makes it possible to experiment with compatible alternative checkpoints.

However, changing checkpoints may require revisiting sampling, conditioning, VAE and LoRA configurations.

### Workflow documentation

The subgraphs could eventually be documented as reusable blueprints, with clear input contracts and tested configurations.

## 20. Architectural Lessons

### 1. Build a stable base before adding complexity

Our original text-to-image workflow became the foundation for all subsequent refinement.

### 2. Keep model configuration centralised

Shared resources reduce inconsistency between generation and detailing.

### 3. Separate prompt interpretation from image processing

Prompt changes and detailing operations solve different classes of problems.

### 4. Preserve reproducibility

Fixed seeds and controlled settings make experiments more meaningful.

### 5. Use the appropriate refinement scope

Whole-image regeneration is not always the best solution to a localised defect.

### 6. Encapsulate meaningful responsibilities

Subgraphs should communicate what a component does, not merely hide arbitrary collections of nodes.

### 7. Preserve observability

Intermediate previews help explain behaviour that would otherwise be hidden.

### 8. Separate architecture from implementation

Our switch migration demonstrated that failing dependencies can be replaced without redesigning the surrounding system.

### 9. Prefer evidence over assumptions

Generated images, execution results and logs are more useful than generic recommendations when diagnosing a specific workflow.

### 10. Allow the workflow to evolve incrementally

The architecture developed through experiments rather than being designed in full at the beginning.

Each successful component became a reusable foundation for the next.

---

## Related Notes

- [[ComfyUI — Understanding Latent Space]]
    
- [[ComfyUI — Noise Seeds and Reproducibility]]
    
- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Automated Detailing]]
    
- [[ComfyUI — Switches and Execution Flow]]
    
- [[ComfyUI — Subgraph Architecture]]
    
- [[Krea 2 — Prompt Engineering]]
    

## Origin

Developed from practical Krea 2 Extended experiments using ComfyUI Desktop on Windows 11.

The workflow began with a Stable Yogi Extended checkpoint, Qwen3-VL text encoding, Qwen Image VAE and a 960 × 1440 text-to-image sampling configuration.

The initial objective was to generate a realistic character portrait.

Subsequent experiments introduced manual masked inpainting, automated eye detailing, SAM segmentation, controlled denoise testing, prompt enhancement, LoRA-loader investigation and conditional output routing.

The processing components were progressively encapsulated into reusable subgraphs.

Following Impact Pack switch problems, the established shared-index routing architecture was retained while the switch implementations were replaced with ComfyUI-Easy-Use equivalents.

The resulting workflow provides a modular, inspectable and extensible foundation for realistic image generation and refinement.

Its most important lesson is that **image-generation workflows benefit from the same engineering disciplines as conventional software: clear responsibilities, stable interfaces, controlled experiments, reproducibility and evidence-led troubleshooting**.