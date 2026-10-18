---

type: reference  
status: active  
created: 2026-10-18  
source: ChatGPT — ComfyUI / Image Generation project  
tags:

- comfyui
    
- wan
    
- wan-2-2
    
- image-to-video
    
- workflow-architecture
    
- lightx2v
    
- sampling
    
- ai-filmmaking
    

---
> [!abstract] Overview  
> This article documents our implementation of **WAN 2.2 A14B Image-to-Video (I2V)** in ComfyUI Desktop, including its dual-expert model architecture, high/low-noise sampling stages, LightX2V acceleration, native/Turbo switching, latent handover and practical troubleshooting.
> 
> Unlike [[WAN 2.2 — Video Generation Fundamentals]], which explains the underlying concepts, this note focuses on how those concepts are implemented in our working ComfyUI workflow.
> 
> Configuration values are recorded from our experiments. They are not universal recommendations for every WAN release or workflow.

## 1. Environment and Scope

### Hardware

|Component|Configuration|
|---|---|
|Operating system|Windows 11|
|Application|ComfyUI Desktop|
|GPU|NVIDIA RTX 5070 Ti|
|GPU memory|16 GB VRAM|
|System memory|32 GB DDR5|
|Python|3.13.12|
|PyTorch|2.12.1+cu130|

### Software baseline

Our last recorded ComfyUI Core version during the experiments was `0.40.0`.

ComfyUI Desktop, Core, Manager and custom nodes have independent version histories. This baseline does not establish their currently installed versions.

The workflow uses native ComfyUI model loading, conditioning, sampling and VAE processing, alongside switching logic for selecting generation modes.

### Workflow objective

Maintain one reusable WAN 2.2 I2V workflow capable of switching between:

- **Native generation:** Higher-step sampling without LightX2V acceleration.
    
- **Turbo generation:** Four-step sampling with LightX2V LoRAs.
    

The intention is to avoid maintaining two separate workflows that gradually diverge in configuration.

---

## 2. WAN 2.2 A14B Model Components

The A14B I2V workflow uses two diffusion model experts.

### Required model files in our template

|Component|Model file|
|---|---|
|High-noise expert|`wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors`|
|Low-noise expert|`wan2.2_i2v_low_noise_14B_fp8_scaled.safetensors`|
|High-noise acceleration|`wan2.2_i2v_lightx2v_4steps_lora_v1_high_noise.safetensors`|
|Low-noise acceleration|`wan2.2_i2v_lightx2v_4steps_lora_v1_low_noise.safetensors`|
|Text encoder|`umt5_xxl_fp8_e4m3fn_scaled.safetensors`|
|VAE|`wan_2.1_vae.safetensors`|

The FP8-scaled diffusion models were selected as part of our 16 GB VRAM configuration.

### Important architectural distinction

The high-noise and low-noise models are not interchangeable.

They are trained for different portions of the denoising trajectory.

Likewise, the LightX2V acceleration LoRAs are expert-specific:

- The high-noise LoRA belongs on the high-noise model.
    
- The low-noise LoRA belongs on the low-noise model.
    

Applying the wrong LoRA to the wrong expert would not represent our intended configuration.

---

## 3. High-Level Workflow Architecture

Our workflow separates model preparation, conditioning, sampling and output processing.

```
                      SOURCE IMAGE
                           |
                           v
                    WanImageToVideo
                           |
              +------------+------------+
              |            |            |
              v            v            v
         Video latent   Positive     Negative
                        conditioning conditioning
              |            |            |
              +------------+------------+
                           |
                           v
                   HIGH-NOISE SAMPLER
                           |
                           v
                   Intermediate latent
                           |
                           v
                    LOW-NOISE SAMPLER
                           |
                           v
                     Final latent
                           |
                           v
                       VAE Decode
                           |
                           v
                       Video frames
                           |
                           v
                     Video output
```

The diagram deliberately omits some individual nodes to make the processing path easier to understand.

The actual workflow also contains:

- Two diffusion model loaders.
    
- Two LightX2V LoRA paths.
    
- Model-selection switches.
    
- Sampling configuration switches.
    
- ModelSamplingSD3 nodes.
    
- Text encoder and VAE loaders.
    
- Frame-count calculation.
    
- Preview and output processing.
    

---

## 4. Model Loading and Preparation

### 4.1 Independent model loaders

The workflow begins with separate diffusion model loaders:

```
Load Diffusion Model — HIGH
            |
            v
High-noise model preparation


Load Diffusion Model — LOW
            |
            v
Low-noise model preparation
```

Each model must remain associated with its intended sampling stage.

### 4.2 ModelSamplingSD3

Our template uses a `ModelSamplingSD3` node on each expert path.

Recorded configuration:

`shift = 5`

Conceptually:

```
High-noise model
       |
       v
ModelSamplingSD3
   shift = 5
       |
       v
High-noise KSampler


Low-noise model
       |
       v
ModelSamplingSD3
   shift = 5
       |
       v
Low-noise KSampler
```

The shift value affects the sampling schedule used by the model.

It should not be treated as an arbitrary image-quality control.

For reproducibility, preserve the known-working shift unless deliberately testing its influence.

---

## 5. Native and Turbo Model Paths

Our workflow includes a boolean control:

`Enable 4steps LoRA?`

This selects between normal model sampling and LightX2V-accelerated sampling.

### 5.1 Native path

In native mode, each expert proceeds without the corresponding LightX2V acceleration LoRA.

```
High-noise model -----------+
                            |
                            v
                      HIGH sampler

Low-noise model ------------+
                            |
                            v
                       LOW sampler
```

### 5.2 Turbo path

In Turbo mode, each expert receives its corresponding LightX2V LoRA.

```
High-noise model
       |
       v
High-noise LightX2V LoRA
       |
       v
HIGH sampler


Low-noise model
       |
       v
Low-noise LightX2V LoRA
       |
       v
LOW sampler
```

### 5.3 Switching logic

The selection control determines which model path reaches each sampler.

Conceptually:

```
                  Enable 4steps LoRA?
                           |
                +----------+----------+
                |                     |
              FALSE                  TRUE
                |                     |
                v                     v
           Native model         LoRA model
                |                     |
                +----------+----------+
                           |
                           v
                      Model switch
                           |
                           v
                        Sampler
```

There are separate model selections for the high-noise and low-noise experts.

> [!important] Keep model paths paired  
> Native mode should use the native sampling configuration and unaccelerated expert models.
> 
> Turbo mode should use the LightX2V expert models and their corresponding four-step sampling configuration.
> 
> Mixing settings unintentionally creates an untested configuration.

---

## 6. Sampling Configuration

Our workflow switches several parameters together.

### Recorded settings

|Parameter|Native|Turbo|
|---|---|---|
|Total steps|20|4|
|CFG|3.5|1.0|
|Split step|10|2|
|High-noise stage|0–10|0–2|
|Low-noise stage|10–20|2–4|
|LightX2V LoRAs|Disabled|Enabled|

These values describe the configuration used in our experiments.

They are not necessarily optimal for every prompt, resolution or future model revision.

### 6.1 Why switch multiple parameters?

The native and accelerated model paths have different sampling expectations.

Switching only the LoRA while leaving the other parameters unchanged would not reproduce our tested Turbo setup.

Therefore, our mode selection also controls:

- Total steps.
    
- CFG.
    
- High/low split point.
    
- Expert model selection.
    

This keeps the two operating modes internally consistent.

---

## 7. High-Noise KSampler Advanced

The first sampler processes the earlier part of the diffusion trajectory.

### Recorded configuration

|Input|Value|
|---|---|
|`add_noise`|enable|
|`start_at_step`|0|
|`end_at_step`|split step|
|`return_with_leftover_noise`|enable|

In native mode, the sampler executes the initial portion of the 20-step trajectory.

In Turbo mode, it executes the initial portion of the four-step trajectory.

### Why enable leftover noise?

The high-noise sampler is not expected to complete generation.

It must return a partially denoised latent suitable for continued processing.

The `return_with_leftover_noise` setting is essential to this handover in our workflow.

If the first sampler fully completes the denoising process, the second stage would not receive the intended intermediate state.

---

## 8. Low-Noise KSampler Advanced

The second sampler continues the denoising trajectory from the high-noise stage.

### Recorded configuration

|Input|Value|
|---|---|
|`add_noise`|disable|
|`start_at_step`|split step|
|`end_at_step`|total steps|
|`return_with_leftover_noise`|disable|

### Why disable add_noise?

The second sampler receives a partially denoised latent from the first sampler.

It must continue processing that latent rather than initialise a fresh noisy sample.

Enabling additional initial noise here would change the intended sampling trajectory.

### Why disable leftover noise?

The low-noise sampler completes the generation process.

Its output should be suitable for decoding into video frames.

### Combined sampling path

```
Initial video latent
        |
        v
+-----------------------+
| HIGH KSampler Advanced|
|                       |
| add_noise: enable     |
| start: 0              |
| end: split            |
| leftover: enable      |
+-----------------------+
        |
        v
Intermediate video latent
        |
        v
+-----------------------+
| LOW KSampler Advanced |
|                       |
| add_noise: disable    |
| start: split          |
| end: total steps      |
| leftover: disable     |
+-----------------------+
        |
        v
Completed video latent
```

> [!warning] Sampling continuity  
> The second sampler must receive the latent produced by the first sampler.
> 
> The two stages are not independent generation branches.

---

## 9. Conditioning and WanImageToVideo

The `WanImageToVideo` node prepares image-conditioned video generation inputs.

Its responsibilities include establishing the requested video dimensions, frame count and image-based conditioning information required by the workflow.

The source image establishes the initial scene appearance.

Text conditioning provides additional guidance for motion, camera behaviour and scene development.

### Conditioning flow

```
Source image --------+
                     |
                     v
              WanImageToVideo
                     |
          +----------+----------+
          |          |          |
          v          v          v
        Latent    Positive    Negative
                  conditioning conditioning
```

The outputs are passed into the appropriate sampling stages.

The source image is not merely a reference thumbnail. It contributes to the initial visual state used by I2V generation.

### Why source-image quality matters

WAN may preserve visible features reasonably well while inventing information that becomes visible during motion.

A character beginning in a crouched pose, for example, may reveal previously hidden clothing and body proportions when standing.

This is why our production pipeline establishes an approved starting keyframe before animation.

See [[AI Filmmaking — Shot Construction and Continuity]].

---

## 10. Duration, FPS and Frame Count

Our initial template used:

`FPS = 16`

Frame count was calculated as:

`length = duration_seconds × FPS + 1`

For a nominal five-second clip:

`5 × 16 + 1 = 81 frames`

The resulting value is passed into the video-generation conditioning path.

### Why calculate frame count?

It allows us to expose duration as a convenient control without manually calculating frame counts.

```
Duration (seconds) ----+
                       |
FPS -------------------+--> Frame calculation
                                |
                                v
                         WanImageToVideo
```

### Important distinction

The frame-count convention belongs to the tested template.

It should not be assumed that all WAN workflows or video encoders use the same formula.

The supported frame counts and dimensions also depend on the model and workflow implementation.

---

## 11. VAE Decoding and Video Output

Once the low-noise sampler finishes, its latent output is decoded using the WAN VAE.

```
LOW sampler
     |
     v
Final video latent
     |
     v
VAE Decode
     |
     v
Sequence of frames
     |
     v
Video encoder / output
```

Our template uses:

`wan_2.1_vae.safetensors`

Although the filename refers to WAN 2.1, it is the VAE supplied for the WAN 2.2 workflow we tested.

The VAE is not an optional enhancement stage. It converts the completed latent representation into viewable frames.

### Preview placement

A preview after VAE decoding allows inspection of the completed generated frames.

This is distinct from attempting to decode the unfinished high-noise latent.

---

## 12. Experiment — Previewing the High-Noise Output

### Hypothesis

Could we bypass the low-noise stage and decode the intermediate high-noise latent to obtain an inexpensive motion preview?

If successful, this might have allowed early rejection of unsuitable motion without completing generation.

### Experimental modification

We temporarily added switching logic after the high-noise sampler.

The idea was to select either:

- The high-noise intermediate latent.
    
- The fully processed low-noise latent.
    

The selected latent would then pass to VAE decoding.

### Turbo test

Configuration:

- Four total steps.
    
- High-noise stage completed two steps.
    
- Low-noise stage bypassed.
    

**Observed result:** Predominantly noise.

### Native test

Configuration:

- Twenty total steps.
    
- High-noise stage completed ten steps.
    
- Low-noise stage bypassed.
    

**Observed result:** Predominantly noise, with only very faint structural information.

Runtime was approximately five minutes.

### Conclusion

The intermediate latent did not provide a usable rough-cut preview.

The low-noise stage is required to produce a viewable result in this tested pipeline.

We removed the experimental branch rather than retaining unnecessary workflow complexity.

> [!tip] Engineering lesson  
> An intermediate computational state is not necessarily a meaningful visual representation.
> 
> Before adding preview branches to a diffusion workflow, establish whether the intermediate output can actually be interpreted by the decoder.

---

## 13. Resolution and Performance Experiments

Our initial source image was a portrait composition of approximately 832×1216.

The first video experiments used a square output of 640×640.

This proved inefficient for the intended shot.

### Recorded performance

|Mode|Resolution|Duration|Runtime|
|---|---|---|---|
|Native|640×640|5 seconds|~10m 40s|
|Turbo|640×640|5 seconds|~96s|
|Native|448×640|5 seconds|~5m 51s|
|Turbo|320×448|5 seconds|~20s|

We also tested Turbo at 224×320, but facial detail was too degraded to be useful for reliable assessment.

### Key findings

**Aspect ratio matters.**

A square video frame was poorly suited to our portrait source image.

**Pixel count matters.**

Moving from 640×640 to 448×640 reduced the number of spatial pixels by approximately 30%.

**Performance improved substantially.**

Native runtime fell from approximately 10m 40s to 5m 51s.

**Lower resolution has practical limits.**

Very small previews become less useful when facial expression and identity are important.

### Working approach

Use a resolution that preserves the intended composition while remaining appropriate for the GPU and the purpose of the generation.

Do not automatically maximise resolution.

---

## 14. Turbo Is Not a Deterministic Native Preview

One of our most important discoveries was that Turbo and native generation can interpret the same source image differently.

### Experiment

We used the same Nate source composition and explored the scene in both Turbo and native modes.

Turbo generated coherent motion quickly.

However, the native result developed:

- A stronger camera push.
    
- A more aggressive facial expression.
    
- Different character movement.
    

The differences were not simply improvements in image detail.

They affected the performance and shot composition.

### Interpretation

LightX2V acceleration changes the sampling process.

The four-step result is therefore not guaranteed to represent the trajectory that the native twenty-step configuration would produce.

### Production implication

Use Turbo to answer questions such as:

- Is the basic action plausible?
    
- Does the model understand the intended scene?
    
- Is this prompt worth exploring?
    
- Are the workflow inputs connected correctly?
    

Do not rely on Turbo alone to establish:

- Exact camera movement.
    
- Final facial performance.
    
- Precise motion timing.
    
- Frame-by-frame continuity with native output.
    

**Native generation must be evaluated as its own take.**

---

## 15. ComfyUI Widget Values and Connected Inputs

During our experiments, we encountered a potentially confusing interface behaviour.

Some exposed or connected KSampler widget values displayed local defaults that did not match the intended mode configuration.

However, the sampler inputs were controlled by upstream switch outputs.

### Why this matters

A connected input can receive its value from another node.

The value displayed in a local widget or retained in a node's configuration is not necessarily the value used during execution.

Conceptually:

```
Mode selector
      |
      v
Steps switch
      |
      v
KSampler steps input
```

The effective sampling value comes from the connected upstream output.

### Troubleshooting approach

When a parameter appears incorrect:

1. Trace the connected input upstream.
    
2. Inspect the switch output.
    
3. Confirm which branch is selected.
    
4. Check the actual value reaching the consuming node.
    
5. Compare execution behaviour with the expected configuration.
    

Do not change a displayed local value without first determining whether it is actually controlling execution.

### Distinguish UI state from runtime state

The graph's visible widget state and its effective execution inputs are related, but they are not always identical.

This is particularly important in workflows with extensive switching and exposed controls.

---

## 16. Switch Nodes and Version Compatibility

Our ComfyUI environment has previously experienced switching-related regressions after updates.

In another workflow, an Impact Pack switch that had previously worked stopped forwarding images correctly after a ComfyUI Core update.

We observed that:

- The input image reached the switch.
    
- The expected output was missing.
    
- An inverse switch worked.
    
- A newly created normal switch worked.
    
- Duplicating a working switch reproduced the failure.
    

These observations were made in the Krea2 workflow, not in WAN itself.

They are included here because our WAN workflow also depends on switch-based configuration selection.

### Diagnostic principle

If a previously working WAN workflow begins behaving unexpectedly after an update, investigate the exact failing node and its connected inputs before changing model settings.

Do not assume that a missing output is caused by the diffusion model.

### Version awareness

When diagnosing switch behaviour, record:

- ComfyUI Core version.
    
- ComfyUI Desktop version.
    
- Relevant custom-node package and version.
    
- Node type.
    
- Expected input and output.
    
- Observed execution behaviour.
    

Avoid replacing entire workflows when a targeted node-level issue can be identified.

See [[ComfyUI — Inpainting and Scene Geometry]] for related workflow experimentation principles.

---

## 17. Controlled Experimentation

Our workflow is intended to support repeatable testing.

A useful experiment changes one meaningful variable while preserving the others.

### Parameters to record

|Category|Parameters|
|---|---|
|Model|High/low expert filenames|
|Acceleration|Native or Turbo|
|LoRAs|Names and strengths|
|Sampling|Steps, CFG, sampler, scheduler, split|
|Conditioning|Prompt, negative prompt, source image|
|Dimensions|Width, height, frame count|
|Randomness|Seed|
|Environment|ComfyUI and relevant node versions|
|Results|Runtime, visual observations, failures|

### Recommended comparison procedure

1. Save the known-working workflow.
    
2. Fix the source image and seed.
    
3. Record the current configuration.
    
4. Change one variable.
    
5. Generate the comparison.
    
6. Inspect the complete sequence.
    
7. Record the observed differences.
    
8. Retain or revert the change based on evidence.
    

### Important qualification

Comparing native and Turbo involves several coordinated changes, including acceleration LoRAs, CFG and step count.

It is therefore a comparison between two complete generation configurations, not an isolated single-variable experiment.

This matters when interpreting the cause of differences.

---

## 18. Troubleshooting by Processing Stage

When generation fails, identify the stage at which the failure occurs.

|Stage|Example symptom|First investigation|
|---|---|---|
|Model loading|Model cannot be loaded|Model path, file, compatibility, memory|
|LoRA loading|Error during model patching|Expert/LoRA pairing and compatibility|
|Conditioning|Invalid or missing conditioning|Source image, text encoder, node outputs|
|Mode selection|Wrong settings reach sampler|Switch inputs and selected branch|
|High-noise sampling|Execution fails early|Model, latent, sampler settings|
|Expert handover|Low stage cannot continue|Latent connection and step boundaries|
|Low-noise sampling|Failure during completion|Model, noise configuration, sampling range|
|VAE decoding|Latent exists but frames unavailable|VAE, latent shape, memory|
|Video output|Frames exist but no video file|Output node and encoding configuration|

This is a diagnostic map, not a recommendation to check every component for every failure.

Start with the actual failing stage.

---

## 19. Production-Oriented Workflow Design

The workflow should support the production process rather than dictate it.

Our developing pipeline is:

```
Animij / still-image generation
                |
                v
        Approved keyframe
                |
                v
         WAN 2.2 I2V
                |
                v
        Generated video shot
                |
                v
       Review / select frames
                |
                v
      Nonlinear video editor
                |
                v
        Completed sequence
```

The starting image may itself be assembled from:

- Character plate.
    
- Environment plate.
    
- Lighting and effects.
    
- Inpainting corrections.
    
- Compositing.
    

These stages should happen before WAN when they establish continuity-critical details.

### Example: Nate at Falco

Our first Nate shot began with the character crouched in Falco's courtyard.

WAN produced approximately three seconds of useful footage before unwanted changes in pose and expression.

Rather than force the model to maintain the crouched pose indefinitely, we decided to use the successful portion as a shot.

Reaction shots can then bridge to a new keyframe in which Nate is already standing.

This reduces the complexity of the motion WAN must generate.

**The video workflow becomes a tool for animating deliberate shots, rather than a machine expected to invent an entire coherent scene.**

---

## 20. Known-Good Configuration Summary

> [!important] Recorded experimental baseline  
> These settings describe our tested WAN 2.2 I2V configuration. Preserve them as a reference when experimenting, but verify installed versions and workflow connections before treating them as a current operational baseline.

|Setting|Native|Turbo|
|---|---|---|
|High-noise model|WAN 2.2 A14B FP8|Same, with high-noise LightX2V|
|Low-noise model|WAN 2.2 A14B FP8|Same, with low-noise LightX2V|
|Steps|20|4|
|CFG|3.5|1.0|
|Split|10|2|
|ModelSamplingSD3 shift|5|5|
|HIGH add_noise|enable|enable|
|HIGH leftover noise|enable|enable|
|LOW add_noise|disable|disable|
|LOW leftover noise|disable|disable|
|FPS|16|16|
|Tested portrait resolution|448×640|320×448|

The exact sampler and scheduler selections should be read from the saved workflow rather than inferred from this table.

---

## 21. Key Engineering Lessons

> [!success] What we established  
> **1. The two WAN experts form one continuous sampling process.** Their latent handover must remain intact.
> 
> **2. The low-noise stage is not merely optional polishing.** Directly decoding the high-noise output did not provide usable preview footage.
> 
> **3. Native and Turbo require coordinated configuration changes.** Model selection, LoRAs, steps, CFG and split must remain consistent.
> 
> **4. Correct aspect ratio can improve both composition and runtime.** More pixels are not automatically better.
> 
> **5. Turbo is useful but not predictive.** It cannot guarantee native camera movement or character performance.
> 
> **6. Connected inputs are authoritative.** A node's displayed widget defaults may not represent its effective execution values.
> 
> **7. Workflow problems and model behaviour are different failure categories.** Diagnose the processing path before changing prompts or models.
> 
> **8. Preserve the known-good architecture.** Experimental branches should earn their place by providing demonstrable value.
> 
> **9. WAN should animate approved visual states.** Character identity, wardrobe and environment continuity should be established upstream wherever practical.

---

## 22. Future Development

Potential improvements to investigate separately:

- Additional motion and camera conditioning.
    
- Reusable shot-configuration controls.
    
- More systematic performance measurements.
    
- Automated generation metadata recording.
    
- Integration with composited character and environment plates.
    
- Output conventions suitable for nonlinear editing.
    
- Reliable character and location continuity across multiple shots.
    

These are development opportunities, not requirements for the current working workflow.

Any new control mechanism should be evaluated for compatibility with the installed WAN model and ComfyUI versions before being incorporated.

---

## 23. Related Notes

- [[WAN 2.2 — Video Generation Fundamentals]] — Model principles, sampling, motion generation and performance.
    
- [[ComfyUI — Inpainting and Scene Geometry]] — Reconstructing environments and correcting source images.
    
- [[ComfyUI — Environment Plates and LoRA Strategy]] — Canonical characters, reusable locations and compositing.
    
- [[AI Filmmaking — Shot Construction and Continuity]] — Shot design, editing, performance and continuity.
    

---

## 24. References

### Upstream resources

- [WAN 2.2 — Official repository](https://github.com/Wan-Video/Wan2.2)
    
- [ComfyUI — Official repository](https://github.com/comfyanonymous/ComfyUI)
    
- [ComfyUI documentation](https://docs.comfy.org/)
    
- [LightX2V — Repository](https://github.com/ModelTC/LightX2V)
    

### Internal experimental sources

The configurations, timings and observations documented here come from our ComfyUI WAN 2.2 experiments, including:

- Initial native and Turbo I2V generation.
    
- Dual-expert latent handover investigation.
    
- High-noise-only VAE decoding experiments.
    
- Portrait-resolution optimisation.
    
- Native-versus-Turbo motion comparison.
    
- Switch-controlled sampling configuration.
    

These measurements are local observations rather than published benchmarks.