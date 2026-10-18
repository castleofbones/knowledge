---

type: reference  
status: active  
created: 2026-10-18  
source:  
tags:

- comfyui
    
- image-generation
    
- diffusion
    
- img2img
    
- inpainting
    
- denoise
    
- krea-2
    

---
> [!abstract] Overview  
> **Image-to-image (img2img)** and **inpainting** both use an existing image as the starting point for diffusion. The fundamental difference is how much of the image is eligible for modification.
> 
> Image-to-image processes the image as a whole, whereas inpainting uses a mask to identify regions for regeneration.
> 
> Understanding this distinction is essential when deciding whether to change an image's overall appearance or repair a specific detail without disturbing the rest of the composition.

## 1. The Fundamental Difference

Both techniques begin with an existing image, encode it into latent space, perform diffusion sampling, and decode the result back into pixels.

The difference lies in how the sampling operation is constrained.

|Property|Image-to-Image|Inpainting|
|---|---|---|
|Starting point|Existing image|Existing image|
|Uses a mask|Not normally|Yes|
|Area eligible for change|Entire image|Mask-designated region|
|Typical purpose|Restyling, reinterpretation, refinement|Repairing or replacing specific content|
|Preservation of unedited pixels|Not guaranteed|Depends on implementation and compositing|
|Denoise|Controls sampling strength|Controls sampling strength within the inpainting process|
|Typical examples|Changing overall lighting or style|Repairing eyes, changing clothing, removing objects|

> [!important] Key principle  
> **Denoise determines the strength of diffusion processing. A mask determines where inpainting is permitted or intended to operate.**
> 
> These are separate controls and should not be confused.

## 2. How Image-to-Image Works

A conventional image-to-image workflow encodes the entire source image into latent space.

The sampler then uses that latent representation as its starting point.

```
Existing IMAGE
      │
      ▼
  VAE Encode ◄──── VAE
      │
      ▼
    LATENT
      │
      ▼
   KSampler ◄──── MODEL
      ▲
      ├────────── Positive Conditioning
      └────────── Negative Conditioning
      │
      ▼
 Modified LATENT
      │
      ▼
  VAE Decode ◄──── VAE
      │
      ▼
 Modified IMAGE
```

The source image provides existing visual structure, while the diffusion model and conditioning influence how the result evolves.

### What is preserved?

Depending on the denoise setting and model behaviour, img2img may preserve:

- General composition
    
- Subject position and pose
    
- Broad colour relationships
    
- Clothing silhouettes
    
- Background structure
    

However, none of these is guaranteed.

Even relatively conservative sampling can change facial details, textures, colours or object geometry.

### What is not guaranteed?

Image-to-image does not provide pixel-perfect preservation of any particular region.

Even where an image looks almost identical, individual pixels may differ because the entire image passes through VAE encoding, diffusion processing and decoding.

If exact preservation of unaffected pixels matters, a separate compositing stage is often appropriate.

See [[ComfyUI — Understanding Latent Space]].

## 3. Understanding Denoise

In ComfyUI's standard `KSampler`, the `denoise` parameter determines how much of the configured sampling schedule is used.

A lower value generally retains more of the source image's structure.

A higher value permits more extensive reinterpretation.

### Conceptual guide

|Denoise|General interpretation|
|---|---|
|1.0|Full sampling schedule; extensive regeneration possible|
|0.7|Significant reinterpretation|
|0.5|Moderate structural and detail changes|
|0.3|Relatively conservative refinement|
|0.1|Very limited diffusion processing|

These values are illustrative, not universal thresholds.

The actual result depends on the model, sampler, scheduler, number of steps, prompt conditioning and source image.

### Denoise is not a percentage

A value of `0.3` does **not** mean:

- Change 30% of the pixels
    
- Preserve exactly 70% of the original image
    
- Modify only 30% of the image's content
    

It controls the diffusion schedule rather than defining a percentage of visual change.

> [!warning] Common misconception  
> Lower denoise does not automatically mean higher quality.
> 
> It may preserve undesirable details, introduce artefacts, or provide too little sampling for the model to produce a coherent correction.

## 4. Our Krea 2 Image-to-Image Experiment

During development of our Krea 2 workflow, we generated a café portrait with a successful overall composition but problematic eyes.

The image already contained many details worth preserving:

- Long blonde hair
    
- Emerald-green eyes
    
- Red lipstick
    
- Dark ribbed turtleneck
    
- Pleated skirt and belt
    
- Crossed legs and tall boots
    
- Coffee cup and café environment
    

Our objective was not to redesign the character.

We wanted to repair the eyes while retaining everything else.

### Initial approach

We introduced an image-to-image branch using the existing generated image as its source.

The image was encoded using the VAE, passed through another KSampler, and decoded.

We experimented with low denoise values, including:

- `0.25`
    
- `0.10`
    

### Observed results

Instead of producing a clean, subtle correction, the output developed undesirable crosshatched textures.

These were particularly visible in:

- Hair
    
- Sweater
    
- Skin
    

The results were less satisfactory than the original image.

### What did this establish?

Our observations established that **whole-image img2img at those settings was not a useful refinement strategy for that image and workflow**.

They did not establish that img2img is inherently unsuitable for Krea 2, or that those denoise values are universally defective.

We had changed the entire image-processing path to correct a small local problem.

That was the architectural mismatch.

### Lesson learned

The image was already satisfactory outside the eyes.

Rather than repeatedly processing the entire image, we needed a way to regenerate only the problematic region.

That led us to inpainting.

## 5. How Inpainting Works

Inpainting introduces a mask to identify the image regions intended for regeneration.

Conceptually:

```
Existing IMAGE ────────┐
                       │
MASK ──────────────────┤
                       ▼
             VAE Encode
            (for Inpainting)
                       ▲
                       │
                      VAE
                       │
                       ▼
                Masked LATENT
                       │
                       ▼
                    KSampler
                       ▲
                       ├──── MODEL
                       ├──── Positive Conditioning
                       └──── Negative Conditioning
                       │
                       ▼
                Refined LATENT
                       │
                       ▼
                  VAE Decode
                       │
                       ▼
                Repaired IMAGE
```

The precise implementation varies between models and inpainting nodes.

Some workflows use latent noise masks, while others prepare masked image content or use model-specific inpainting conditioning.

The important principle is that the mask provides spatial information about where regeneration is desired.

### Mask interpretation

For the mask convention used in our experiment:

- White identifies the region to be regenerated.
    
- Black identifies the region intended to remain unaffected.
    
- Intermediate values can represent partial mask strength or softened transitions.
    

The exact treatment of these values depends on the receiving node.

## 6. Our Manual Eye-Inpainting Experiment

We replaced the unsuccessful whole-image refinement approach with a masked repair workflow.

### Step 1 — Create the mask

Using ComfyUI's `Load Image` mask editor, we painted two white regions over the eyes.

The masks covered the problematic eye structures, including the eyelids and eyelashes, while avoiding the eyebrows.

The mask preview showed two white shapes against a black background.

The original image dimensions were:

`960 × 1440`

### Step 2 — Encode for inpainting

We used:

`VAE Encode (for Inpainting)`

Inputs:

|Input|Source|
|---|---|
|`pixels`|Original image|
|`mask`|Painted eye mask|
|`vae`|Existing Krea 2 VAE|
|`grow_mask_by`|`6`|

The mask-growth parameter expanded the mask used by the encoding operation.

This provided a little additional area around the target regions for regeneration.

### Step 3 — Sample the masked latent

We connected the inpainting latent to a new KSampler.

Our test configuration was:

|Parameter|Value|
|---|---|
|Steps|12|
|CFG|1.5|
|Sampler|Euler|
|Scheduler|Simple|
|Denoise|1.0|
|Model|Existing Krea 2 model path|
|Conditioning|Existing positive and negative conditioning|

### Why use denoise 1.0?

The original eyes contained structural problems.

A very low denoise value could leave too much of that problematic structure intact.

Using `denoise = 1.0` allowed substantial regeneration within the inpainting process.

This was a deliberate choice for our experiment, not a universal recommendation for all inpainting tasks.

### Step 4 — Decode the result

The sampled latent was passed through `VAE Decode`.

The resulting image showed substantially improved eye anatomy.

However, we still wanted explicit control over which final pixels were retained.

That required compositing.

## 7. Why We Added Pixel Compositing

Inpainting and compositing perform different jobs.

**Inpainting generates replacement content.**

**Compositing determines which replacement pixels appear in the final image.**

We used:

`ImageCompositeMasked`

With these connections:

|Input|Source|
|---|---|
|`destination`|Original image|
|`source`|Decoded inpainting result|
|`mask`|Original painted mask|
|`x`|0|
|`y`|0|
|`resize_source`|false|

Conceptually:

```
Original IMAGE ────────────────┐
                               │
Repaired IMAGE ────────────────┤
                               ▼
                       ImageCompositeMasked
                               ▲
                               │
                         Original MASK
                               │
                               ▼
                         Final IMAGE
```

This allowed the final image to retain the original pixels outside the composite mask.

### Why use the original mask?

The original painted mask represented the exact region we had chosen to replace.

The mask used internally during inpainting could be grown or otherwise processed.

By supplying the original mask to the final composite, we controlled the visible replacement boundary independently of the inpainting preparation.

> [!important] Architectural insight  
> The mask used to guide generation and the mask used to composite the final image do not have to be identical.
> 
> This separation provides greater control over the final output.

See [[ComfyUI — Masks and Compositing]] for a more detailed treatment.

## 8. Integrating Inpainting Into the Main Workflow

Our first experiment used a separately loaded image.

Once the repair worked, we integrated it into the existing text-to-image pipeline.

The original `VAE Decode` output was connected directly to the inpainting encoder's `pixels` input.

The `Load Image` node remained only to supply the manually painted mask.

```
                  Base KSampler
                        │
                        ▼
                   VAE Decode
                        │
                        ├───────────────────┐
                        │                   │
                        ▼                   │
                VAE Encode                  │
               (for Inpainting)             │
                        ▲                   │
                        │                   │
                 Painted MASK               │
                        │                   │
                        ▼                   │
                Inpaint KSampler            │
                        │                   │
                        ▼                   │
                   VAE Decode               │
                        │                   │
                        ▼                   ▼
                   ImageCompositeMasked
                            │
                            ▼
                        Final IMAGE
```

This eliminated the need to reload the original image for every test.

### Why the fixed seed mattered

The text-to-image generation used a fixed seed.

With the same prompt, model and settings, the base image was reproducible.

That meant the painted eye mask continued to align with the same image.

However, if the prompt, composition or generation settings changed, the mask could become misaligned.

A fixed seed does not guarantee that the eyes remain in the same location after other generation parameters change.

### Limitation of manual masks

Manual masks are tied to the geometry of a particular image.

They are excellent for deliberate repairs but less suitable for arbitrary new images where the target feature may move.

This limitation motivated our subsequent exploration of automated detection and detailing.

See [[ComfyUI — Automated Detailing]].

## 9. When to Choose Each Technique

### Choose image-to-image when:

- The entire image is eligible for reinterpretation.
    
- You want to change the overall visual style.
    
- You want to explore variations of an existing composition.
    
- You are comfortable with some changes outside the primary area of interest.
    
- Pixel-perfect preservation is not required.
    

### Choose inpainting when:

- A specific feature needs repairing.
    
- You want to replace an object or clothing detail.
    
- Most of the original image is already satisfactory.
    
- The surrounding composition should remain stable.
    
- You need spatial control over regeneration.
    

### Add compositing when:

- You want explicit control over the final replacement region.
    
- Original pixels outside that region must be preserved.
    
- The generated repair is satisfactory but surrounding areas have changed.
    
- You need to separate generation boundaries from final visible boundaries.
    

## 10. Common Failure Modes

|Symptom|Possible explanation|Investigation|
|---|---|---|
|Whole image changes unexpectedly|Img2img processes the entire latent|Consider masked inpainting|
|Target detail barely changes|Insufficient effective regeneration|Test a higher denoise value|
|Repaired region looks unrelated|Excessive regeneration or inadequate context|Review denoise, crop and conditioning|
|Visible boundary around repair|Hard mask edge or mismatch between images|Inspect mask and compositing transition|
|Repair appears in the wrong place|Mask no longer aligns with source|Preview the mask against the current image|
|Textures become distorted|Model/sampling/VAE behaviour|Compare against the original and change one variable|
|Unmasked pixels differ|Final image uses a decoded/generated output rather than original pixels|Composite onto the original image|

These are diagnostic starting points rather than guaranteed explanations.

## 11. Lessons From Our Experiment

### 1. Match the operation to the scope of the problem

A small local defect does not necessarily justify regenerating an entire image.

### 2. Denoise and masks solve different problems

Denoise controls sampling strength.

Masks provide spatial constraints.

### 3. Preserve successful content deliberately

If most of an image is already correct, there is value in protecting it from unnecessary processing.

### 4. Separate generation from final presentation

A generated repair and the final composite are distinct outputs with different responsibilities.

### 5. Test incrementally

Our controlled experiments made it possible to identify why whole-image refinement was unsuitable and verify that masked regeneration produced better results.

### 6. Manual and automatic detailing are complementary

Manual masks provide precise artistic control.

Automated detectors provide convenience and adaptability across changing compositions.

Neither completely replaces the other.

## 12. Practical Decision Guide

```
Do I already have an image?
             │
             ▼
            Yes
             │
             ▼
Should the entire image be
eligible for modification?
             │
       ┌─────┴─────┐
       │           │
      Yes          No
       │           │
       ▼           ▼
     Img2Img    Inpainting
                   │
                   ▼
          Is exact preservation
          outside the edit needed?
                   │
              ┌────┴────┐
              │         │
             Yes        No
              │         │
              ▼         ▼
         Composite    Inspect
         onto source   result
```

This is a conceptual decision guide. The best implementation may differ depending on the model and available nodes.

---

## Related Notes

- [[ComfyUI — Understanding Latent Space]]
    
- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Automated Detailing]]
    
- [[ComfyUI — Switches and Execution Flow]]
    
- [[ComfyUI — Subgraph Architecture]]
    
- [[Krea 2 — Prompt Engineering]]
    
- [[Krea 2 — Practical Workflow Architecture]]
    

## Origin

Based on our practical Krea 2 Extended experiments in ComfyUI Desktop.

The original problem was malformed eyes in an otherwise successful café portrait.

Whole-image img2img experiments at low denoise produced undesirable texture artefacts. A manual eye mask, `VAE Encode (for Inpainting)`, a second KSampler and `ImageCompositeMasked` produced a substantially better result.

The resulting masked-repair implementation became the foundation of our reusable **Masked Detailer** subgraph.

The general principles apply beyond Krea 2, while the recorded node settings and results describe our particular workflow.