---
type: reference
status: active
created: 2026-10-18
source:
tags:
  - comfyui
  - image-generation
  - diffusion
  - latent-space
  - vae
  - krea-2
---
> [!abstract] Overview  
> ComfyUI image-generation workflows operate with two fundamentally different representations of an image: **pixel images** and **latent representations**. Understanding the distinction explains why samplers, VAE encoders, VAE decoders, inpainting and image-to-image generation work the way they do.

## 1. Pixel Space vs Latent Space

### Pixel space

Pixel space is the conventional representation of an image: a grid of pixels containing colour information.

In ComfyUI, the `IMAGE` data type represents image data that nodes can display, save, crop, composite or otherwise manipulate.

Typical nodes operating in pixel space include:

- `Load Image`
    
- `Preview Image`
    
- `Save Image`
    
- `ImageCompositeMasked`
    

A 960 × 1440 RGB image contains 1,382,400 pixels, each with three colour channels.

### Latent space

Latent space is a learned numerical representation of an image. Instead of explicitly storing the RGB values of every pixel, it represents visual information in a form that the diffusion model can process.

In ComfyUI, this is generally passed between nodes as the `LATENT` data type.

A latent representation is not directly viewable as an ordinary image. It must be decoded using a compatible VAE.

Importantly, latent representations are **model-family-dependent**. Their channel counts, spatial dimensions, scaling and interpretation are not universal.

> [!important] Key distinction  
> `IMAGE` describes pixels that can be displayed.
> 
> `LATENT` carries the representation that the diffusion model processes.
> 
> They are not interchangeable, even when both correspond to the same intended image.

## 2. The Role of the VAE

VAE stands for **Variational Autoencoder**.

In diffusion workflows, its two principal operations are encoding and decoding.

### VAE Encode

Converts an image from pixel space into a latent representation.

```
IMAGE
  │
  ▼
VAE Encode ◄── VAE
  │
  ▼
LATENT
```

This is useful when an existing image needs to become the starting point for diffusion processing.

### VAE Decode

Converts a latent representation back into a pixel image.

```
LATENT
  │
  ▼
VAE Decode ◄── VAE
  │
  ▼
IMAGE
  │
  ▼
Save Image
```

The VAE is learned, so encoding and decoding are not necessarily perfectly lossless. Small differences in fine detail can occur.

### Why VAE compatibility matters

The VAE must understand the latent representation used by the model.

A VAE intended for one architecture cannot be assumed compatible with another.

In our Krea 2 workflow, we used:

`qwen_image_vae.safetensors`

This is part of the model-specific configuration rather than a universal ComfyUI setting.

## 3. What the KSampler Actually Does

The KSampler operates in latent space.

Its main inputs include:

|Input|Purpose|
|---|---|
|`model`|The diffusion model performing denoising|
|`positive`|Positive conditioning|
|`negative`|Negative conditioning, where applicable|
|`latent_image`|Initial latent data|
|`seed`|Controls the random noise generation|
|`steps`|Number of sampling steps|
|`cfg`|Classifier-free guidance setting|
|`sampler_name`|Sampling algorithm|
|`scheduler`|Noise-level schedule|
|`denoise`|Amount of the diffusion schedule used|

The sampler progressively transforms a noisy latent into a representation corresponding to an image.

The resulting latent still needs VAE decoding before it becomes a viewable image.

### Our Krea 2 baseline

|Parameter|Value|
|---|---|
|Image dimensions|960 × 1440|
|Steps|12|
|CFG|1.5|
|Sampler|Euler|
|Scheduler|Simple|
|Denoise|1.0|
|Model sampling shift|3|

These are **settings from our working experiment**, not universal recommendations for every Krea 2 checkpoint.

## 4. Text-to-Image Generation

Text-to-image generation typically begins with an empty latent allocation.

An `Empty Latent Image` node establishes the desired dimensions and batch size. It does not contain an existing photograph or rendered scene.

The sampler uses that latent allocation and a seeded noise process to generate an image according to the model and conditioning.

```
Empty Latent Image
        │
        ▼
     KSampler ◄──── MODEL
        ▲
        ├────────── Positive Conditioning
        └────────── Negative Conditioning
        │
        ▼
    VAE Decode ◄──── VAE
        │
        ▼
       IMAGE
        │
        ▼
    Save Image
```

### What does denoise 1.0 mean here?

For ordinary text-to-image generation, `denoise = 1.0` uses the full configured sampling schedule.

Because there is no existing image to preserve, this is the normal starting point.

### What does a fixed seed accomplish?

A fixed seed allows repeatable experiments when the remaining conditions are unchanged.

However, a seed does **not** guarantee an identical composition after changing the prompt, model, resolution, sampler or other generation parameters.

It controls the random starting conditions, not the finished image's identity.

## 5. Image-to-Image Generation

Image-to-image generation starts with an existing pixel image.

That image must first be encoded into latent space.

```
Load Image
    │
    ▼
VAE Encode ◄──── VAE
    │
    ▼
  LATENT
    │
    ▼
 KSampler ◄──── MODEL + CONDITIONING
    │
    ▼
VAE Decode ◄──── VAE
    │
    ▼
  IMAGE
```

Unlike text-to-image, the sampler now has an existing image representation to work from.

### The significance of denoise

In ComfyUI's standard KSampler, the denoise parameter controls how much of the sampling schedule is applied.

Broadly:

|Denoise|Typical behaviour|
|---|---|
|1.0|Extensive regeneration; weak preservation of the starting image|
|0.7|Significant changes while retaining some initial structure|
|0.3|More conservative refinement|
|0.1|Very limited diffusion-driven change|

These are qualitative descriptions, not guaranteed outcomes. Actual behaviour depends on the model, image, conditioning and sampling configuration.

### Our failed experiment

We attempted whole-image refinement of our café portrait using low denoise values, including 0.25 and 0.10.

Instead of obtaining clean, subtle improvements, we observed undesirable crosshatched textures in the hair, clothing and skin.

The important lesson was not that low-denoise image-to-image is inherently defective.

It was that **low denoise does not guarantee visually clean or useful refinement**, particularly when the model and workflow are not well suited to that operation.

This motivated our move toward localised inpainting.

## 6. Latent Inpainting

Inpainting allows diffusion processing to concentrate on selected areas rather than treating the entire image as equally editable.

Our manual eye-repair workflow used:

`VAE Encode (for Inpainting)`

This node accepted:

- The original pixel image
    
- A mask identifying the target region
    
- The VAE
    
- A mask-growth setting
    

The resulting latent carried information used to constrain the inpainting operation.

Our experiment used `grow_mask_by = 6`, followed by a KSampler at `denoise = 1.0`.

That high denoise setting was useful because we wanted to regenerate the problematic eyes rather than merely make tiny adjustments to their existing malformed structure.

**The crucial distinction:** denoise controls the sampling process; the mask controls which image regions are designated for inpainting.

The exact handling of masked and unmasked regions depends on the nodes and implementation.

We then used `ImageCompositeMasked` to place the repaired result over the original image, preserving the original pixels outside the chosen composite mask.

See [[ComfyUI — Masks and Compositing]] and [[ComfyUI — Masks and Compositing]].

## 7. Why Latent and Pixel Operations Must Be Kept Distinct

Consider three different operations:

**Changing an image through diffusion**

Requires a suitable latent representation and a sampling operation.

**Changing visible pixels directly**

Can be performed using image-processing nodes without diffusion.

**Replacing a small region with a generated repair**

May involve both latent-space generation and pixel-space compositing.

This distinction explains why our eye-repair workflow had two separate stages:

1. Generate corrected image content through masked diffusion.
    
2. Composite the corrected pixels onto the original image.
    

The sampler generated the replacement; the composite controlled the final visible replacement region.

These operations solve different problems.

## 8. Mental Model: A Compiler Pipeline

A useful software-engineering analogy is to think of the workflow as a processing pipeline with distinct data representations.

```
Text Prompt
    │
    ▼
Text Encoder
    │
    ▼
CONDITIONING ───────────┐
                        │
Initial LATENT ───────► Sampler ◄── MODEL
                        │
                        ▼
                  Refined LATENT
                        │
                        ▼
                    VAE Decode
                        │
                        ▼
                      IMAGE
                        │
                        ▼
                  Save / Display
```

The prompt encoder converts textual instructions into conditioning.

The sampler operates on latent data under that conditioning.

The VAE decoder converts the resulting latent into visible pixels.

Image-processing nodes then operate on the decoded result.

This is an analogy, not a claim that latent space is literally an intermediate programming language. Its value is in making **data types, transformations and boundaries** explicit.

## 9. Common Misconceptions

### “The sampler generates a PNG.”

Not directly. It produces latent output, which must be decoded before conventional image saving.

### “Denoise 0.1 means changing exactly 10% of the image.”

No. Denoise is not a percentage of pixels changed.

### “A fixed seed preserves the same character.”

Not necessarily. It helps with repeatability under matching conditions, but it does not establish persistent character identity.

### “A latent is simply a smaller RGB image.”

No. Latents are learned representations, not ordinary downscaled RGB images.

### “Inpainting and compositing are the same thing.”

No. Inpainting generates or modifies content; compositing combines pixel images according to a mask.

## 10. Practical Takeaways

- Understand the data type entering and leaving every node.
    
- Keep track of when the workflow moves between `IMAGE` and `LATENT`.
    
- Treat the VAE as part of the model architecture.
    
- Use fixed seeds when comparing controlled experiments.
    
- Do not assume low denoise automatically produces better detail.
    
- Prefer localised operations when only a small part of an otherwise successful image needs repairing.
    
- Separate content generation from final pixel compositing when exact preservation matters.
    

---

## Related Notes

- [[ComfyUI — Noise Seeds and Reproducibility]]
    

- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Automated Detailing]]
    
- [[ComfyUI — Switches and Execution Flow]]
    
- [[ComfyUI — Subgraph Architecture]]
    
- [[Krea 2 — Prompt Engineering]]
    
- [[Krea 2 — Practical Workflow Architecture]]
    

## Origin

Developed from practical experimentation with a Krea 2 Extended workflow in ComfyUI Desktop, including text-to-image generation, unsuccessful whole-image refinement, successful masked eye repair and subsequent automated detailing.

The architecture and conceptual explanations are intended to remain useful across compatible diffusion-model workflows, while the recorded Krea 2 settings describe the specific experiment rather than universal defaults.