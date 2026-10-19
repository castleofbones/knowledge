---
type: reference
status: active
created: 2026-10-18
source:
tags:
  - comfyui
  - image-generation
  - automated-detailing
  - face-detailer
  - eye-detailer
  - ultralytics
  - yolo
  - sam
  - inpainting
  - krea-2
purpose: knowledge
---
> [!abstract] Overview  
> Automated detailing uses object detection, optional segmentation, and targeted diffusion processing to improve selected regions of a generated image.
> 
> Rather than manually painting a mask for every image, a detailing workflow can identify relevant features, extract a region for processing, regenerate its content, and integrate the result into the original image.
> 
> This article explores the distinction between detection and segmentation, the roles of YOLO and SAM, crop sizing, denoise, sampling, and the reusable **Eye Detailer** subgraph developed during our Krea 2 experiments.

## 1. Why Automated Detailing?

Our original Krea 2 café portrait had a successful overall composition but problematic eyes.

We initially attempted whole-image image-to-image refinement. This produced undesirable texture artefacts without reliably solving the problem.

We then created a manual masked-inpainting workflow, which successfully repaired the eyes.

However, the manual approach had an important limitation.

**The painted mask was tied to the geometry of a particular image.**

If the character moved, the framing changed, or the face appeared at a different position, the mask could become misaligned.

Automated detailing addresses this limitation by detecting relevant image features dynamically.

### Manual vs automated detailing

|Characteristic|Manual detailing|Automated detailing|
|---|---|---|
|Region selection|User-painted mask|Detector-generated region|
|Adaptability|Tied to a particular composition|Can adapt to changing compositions|
|Precision|Direct artistic control|Depends on detector and segmentation quality|
|Setup|Simple for individual repairs|More complex initial configuration|
|Repeatability|Good for fixed images|Useful across batches and variations|
|Typical application|Specific corrections|Recurring feature refinement|

Neither approach universally replaces the other.

Our final workflow retained both.

See [[ComfyUI — Masks and Compositing]] and [[ComfyUI — Masks and Compositing]].

## 2. The Automated Detailing Pipeline

An automated detailer combines several responsibilities.

Conceptually:

```
              Original IMAGE
                    │
                    ▼
             Object Detection
                    │
                    ▼
              Bounding Box
                    │
                    ▼
             Crop / Resize
                    │
                    ▼
          Optional Segmentation
                    │
                    ▼
            Targeted Sampling
                    │
                    ▼
           Restore / Composite
                    │
                    ▼
              Refined IMAGE
```

This diagram describes the logical stages, not a guaranteed internal execution order for every implementation.

Different detailers may perform segmentation, cropping, resizing, masking, and compositing in different sequences.

The important point is that automated detailing combines **region discovery** with **localised image generation**.

## 3. Detection vs Segmentation

Detection and segmentation answer different questions.

### Object detection

Object detection identifies the presence and approximate location of an object or feature.

Its output commonly includes a bounding box.

For example, a face detector may identify a rectangular region containing a person's face.

```
┌──────────────────────────────┐
│                              │
│       Original Image         │
│                              │
│       ┌──────────────┐       │
│       │              │       │
│       │     Face     │       │
│       │              │       │
│       └──────────────┘       │
│                              │
└──────────────────────────────┘
```

A bounding box describes the region's location and extent.

It does not necessarily identify the precise pixels belonging to the object.

### Segmentation

Segmentation identifies a more detailed spatial region.

Instead of merely returning a rectangle, a segmentation model can produce a mask approximating the shape of the detected object.

For example, a face segmentation mask may follow the face's outline rather than including the entire rectangular background around it.

### Comparison

|Detection|Segmentation|
|---|---|
|Identifies objects or features|Identifies their spatial regions|
|Often outputs bounding boxes|Outputs masks|
|Useful for locating targets|Useful for refining selection boundaries|
|May include background inside the box|Can exclude more surrounding background|
|Provides coarse spatial information|Provides finer spatial information|

> [!important] Key distinction  
> **Detection determines where a feature is. Segmentation determines which pixels belong to the selected region.**
> 
> A detailer may use detection alone or combine detection with segmentation.

## 4. YOLO and Ultralytics

YOLO stands for **You Only Look Once**.

It refers to a family of object-detection architectures designed to identify objects in images.

Ultralytics provides implementations and tooling for several YOLO model families.

In ComfyUI, compatible custom-node packages can expose YOLO detectors to other nodes.

### Our detector configuration

We used:

`UltralyticsDetectorProvider`

With the model:

`bbox/face_yolov9c.pt`

The provider's `bbox_detector` output was connected to the FaceDetailer node.

Conceptually:

```
UltralyticsDetectorProvider
          │
          │ bbox_detector
          ▼
      FaceDetailer
```

The detector identified the face region for subsequent detailing.

### What does the model filename mean?

The filename identifies the detector checkpoint.

In our case, it was a face-detection model based on YOLOv9.

The `bbox/` prefix identified its location within the detector model selection used by the workflow.

This should not be confused with the main Krea 2 image-generation checkpoint.

The two models have different responsibilities:

- **Krea 2 checkpoint:** Generates or refines image content.
    
- **YOLO detector:** Locates the face region.
    

The detector does not itself regenerate the eyes, face, or other image details.

## 5. The FaceDetailer Node

Our experiment used:

`FaceDetailer`

From the Impact Pack ecosystem.

The node combines detection and diffusion-based refinement into a higher-level operation.

### Principal inputs

|Input|Responsibility|
|---|---|
|`image`|Original pixel image|
|`model`|Diffusion model used for refinement|
|`clip`|Text-encoding resource|
|`vae`|Compatible VAE|
|`positive`|Positive conditioning|
|`negative`|Negative conditioning|
|`bbox_detector`|Detector used to locate the target|
|`sam_model_opt`|Optional SAM segmentation model|

The node also exposes sampling, detection, cropping, masking, and other configuration parameters.

The precise available controls and internal behaviour depend on the installed Impact Pack version.

### Our initial wiring

We connected:

- The base image from `VAE Decode`
    
- The existing post-shift Krea 2 model
    
- The existing CLIP and VAE
    
- The existing positive and negative conditioning
    
- The Ultralytics face detector
    

This allowed us to investigate automatic detailing without changing the main generation architecture.

## 6. Bounding Boxes and Crop Factors

Once the detector identifies a face, the detailer can extract a region for processing.

The size of this region affects how much surrounding context is available.

Our FaceDetailer configuration exposed:

`bbox_crop_factor`

### What does the crop factor influence?

Broadly, it controls how much area around the detected bounding box is included in the processing crop.

A larger value provides more surrounding context.

A smaller value focuses more tightly on the detected feature.

### Our experiments

We compared two values.

|`bbox_crop_factor`|Observed cropped-refined preview|
|---|---|
|3.0|Approximately 730 × 1065|
|2.0|Approximately 487 × 710|

The smaller crop factor produced a tighter processing region.

We preferred `2.0` for our particular portrait.

### Why crop size matters

A crop that is too tight may omit useful contextual information.

For example, refining eyes may benefit from information about the surrounding face, eyelids, eyebrows, skin, and lighting.

A crop that is too large may reduce the relative amount of sampling attention given to the feature requiring correction.

It may also expose more surrounding content to modification, depending on masking and implementation.

> [!tip] Practical principle  
> Crop sizing is a balance between **local detail** and **surrounding context**.
> 
> More context is not automatically better, and a tighter crop is not automatically more precise.

## 7. Guide Size and Processing Resolution

Our FaceDetailer configuration included:

- `guide_size = 512`
    
- `guide_size_for = bbox`
    
- `max_size = 1024`
    

These parameters influence how the detailer determines its working resolution.

The detected region may be resized for processing rather than sampled at its original pixel dimensions.

### Why process a crop at a different resolution?

A face occupying a relatively small part of a full image may contain too few pixels for effective local refinement.

Processing the cropped region at a larger working resolution can give the diffusion model more spatial detail to work with.

The result is then integrated back into the original image.

### Important distinction

A detailer's working resolution is not necessarily the same as:

- The original image resolution
    
- The detected bounding-box dimensions
    
- The final output resolution
    
- The dimensions of a diagnostic preview
    

The exact resizing calculation depends on the node implementation.

Our observed crop dimensions demonstrated that the crop-factor setting affected the working region, but they do not by themselves establish every internal resizing step.

## 8. Introducing SAM

SAM stands for **Segment Anything Model**.

It is a segmentation architecture designed to produce masks from image content and spatial prompts.

We introduced SAM after successfully running FaceDetailer with bounding-box detection.

### Our configuration

We used:

`SAMLoader (Impact)`

With:

`sam_vit_b_01ec64.pth`

And:

`device_mode = AUTO`

The loader's output was connected to:

`FaceDetailer → sam_model_opt`

### What changed?

Before connecting SAM, the mask preview showed a rectangular selection associated with the detected face region.

After connecting SAM, the mask preview changed to a face-shaped region.

This was direct evidence that segmentation was affecting the selected area.

### What SAM contributed

YOLO supplied the detected face location.

SAM provided a more detailed spatial selection.

Conceptually:

```
             Original IMAGE
                    │
                    ▼
              YOLO Detector
                    │
                    ▼
              Face Bounding Box
                    │
                    ▼
                 SAM Model
                    │
                    ▼
              Segmentation Mask
                    │
                    ▼
             Detailing Process
```

This is a conceptual relationship, not a claim about the exact internal call order.

### What SAM did not do

SAM did not independently generate better eyes.

It supplied segmentation information used by the detailing workflow.

The diffusion model and sampling operation remained responsible for generating the refined image content.

## 9. SAM Masks Are Not Necessarily Eye Masks

This distinction became important during our investigation.

We were trying to improve the eyes, but our detector was a **face detector**, not an eye detector.

The SAM mask we observed was associated with the face region.

It was not evidence of automatic eye-specific segmentation.

### Why the distinction matters

A workflow can produce better eyes without exclusively processing the eyes.

For example, a face-level refinement may improve:

- Eye anatomy
    
- Eyelids
    
- Eyelashes
    
- Skin texture
    
- Facial proportions
    
- Other facial details
    

The resulting improvement does not establish that the mask was restricted to the eyes.

### Our diagnostic preview

We exposed the FaceDetailer `cropped_refined` output to inspect the processed region.

At one point, a preview appeared with dimensions of approximately:

`379 × 118`

Those dimensions alone did not prove that SAM had generated an eye-only mask.

A cropped preview and a segmentation mask represent different information.

> [!warning] Avoid this inference  
> A preview focused on the eyes does not necessarily mean the underlying segmentation mask selects only the eyes.
> 
> To establish the actual selected region, inspect the mask output itself.

## 10. Denoise in Automated Detailing

Automated detailing still relies on diffusion sampling.

The denoise parameter therefore remains important.

However, the detailer's crop and mask determine the context in which sampling takes place.

### Our initial configuration

We began with:

`denoise = 0.50`

This produced substantially improved eyes, but the face was also noticeably re-rendered.

The result was technically interesting but changed more of the character's appearance than we wanted.

### Our refinement

We reduced denoise to:

`0.30`

This produced a better balance for our portrait.

The eyes improved while the character's facial identity and surrounding appearance were more closely preserved.

### Observed comparison

|Denoise|Our observation|
|---|---|
|0.50|Stronger facial regeneration; improved eyes but greater changes to appearance|
|0.30|More conservative refinement; improved eyes with better preservation|

These are observations from our workflow, not universal FaceDetailer recommendations.

### Why not simply use denoise 1.0?

Our manual eye-inpainting experiment used denoise 1.0 because we had explicitly painted the defective eye regions and wanted substantial regeneration there.

The automated detailer was operating with a face-level detection and segmentation process.

Using a high denoise value could therefore produce more extensive changes than desired.

The appropriate denoise value depends on the **scope of the selected region** as well as the required correction.

## 11. Comparing Manual and Automated Eye Repair

Our experiments produced two successful approaches.

### Manual Masked Detailer

Uses:

- A manually painted mask
    
- `VAE Encode (for Inpainting)`
    
- A dedicated KSampler
    
- `VAE Decode`
    
- `ImageCompositeMasked`
    

Advantages:

- Precise selection
    
- Deliberate control over the replacement area
    
- Explicit preservation of original pixels outside the composite mask
    

Limitations:

- Requires manual mask creation
    
- Mask alignment depends on the source composition
    
- Less convenient for arbitrary new generations
    

### Automated Eye Detailer

Uses:

- YOLO face detection
    
- Optional SAM segmentation
    
- FaceDetailer processing
    
- Localised diffusion refinement
    

Advantages:

- Automatically locates the face
    
- Adapts to changing compositions
    
- Requires no manually painted eye mask
    
- Convenient for repeated generations
    

Limitations:

- The selected region is not necessarily limited to the eyes
    
- Detection and segmentation quality affect the result
    
- Higher denoise can alter facial identity
    
- Processing behaviour depends on the detailer implementation
    

### Which approach is better?

Neither is universally superior.

Our automated approach became the convenient default for recurring eye improvements.

The manual approach remained available for deliberate, targeted repairs.

## 12. The Eye Detailer Subgraph

Once the automated configuration was working satisfactorily, we encapsulated it into a reusable subgraph.

Initially, the subgraph was named:

`Face Detailer`

We subsequently renamed it:

`Eye Detailer`

### Why rename it?

Although the implementation uses face detection, its intended role in our workflow is to improve eye quality.

The new name describes the **purpose of the abstraction** rather than merely the underlying detector.

This is an intentional architectural choice.

It does not imply that the implementation uses a dedicated eye detector or eye-only segmentation.

### Subgraph inputs

Our reusable detailer exposed the relevant generation resources and parameters.

|Input|Purpose|
|---|---|
|`IMAGE`|Base image to refine|
|`MODEL`|Diffusion model|
|`CLIP`|Text encoder resource|
|`VAE`|Compatible VAE|
|Positive conditioning|Positive prompt conditioning|
|Negative conditioning|Negative prompt conditioning|
|Seed|Sampling seed|
|Steps|Sampling step count|
|CFG|Guidance value|
|Denoise|Refinement strength|
|Detector model name|YOLO detector selection|
|SAM model name|Segmentation model selection|

The exact external socket labels depend on the saved subgraph interface.

### Internal components

Conceptually:

```
                 IMAGE
                   │
                   ▼
              FaceDetailer ◄──── MODEL
                   ▲
                   ├────────── CLIP
                   ├────────── VAE
                   ├────────── CONDITIONING
                   │
       ┌───────────┴───────────┐
       │                       │
       ▼                       ▼
UltralyticsDetectorProvider  SAMLoader
       │                       │
       └───────────┬───────────┘
                   │
                   ▼
             FaceDetailer
                   │
                   ▼
             Refined IMAGE
```

The diagram illustrates the relationships among the components rather than literal duplicate FaceDetailer nodes.

### Our selected configuration

|Parameter|Value|
|---|---|
|Detector|`face_yolov9c.pt`|
|SAM model|`sam_vit_b_01ec64.pth`|
|SAM device mode|AUTO|
|Steps|12|
|CFG|1.5|
|Sampler|Euler|
|Scheduler|Simple|
|Denoise|0.30|
|Guide size|512|
|Guide size basis|bbox|
|Maximum size|1024|
|Bounding-box crop factor|2.0|

These values describe the working configuration from our experiment.

They should be treated as a reproducible starting point for that workflow, not as universal defaults.

## 13. Diagnostic Outputs and Observability

One of our architectural decisions was to expose a diagnostic preview from inside the subgraph.

We connected the FaceDetailer `cropped_refined` output to a subgraph output so that it could be inspected from the parent workflow.

### Why this was useful

It allowed us to see what the detailing process produced without opening the subgraph.

This supported:

- Visual verification of the processed crop
    
- Comparison between denoise settings
    
- Investigation of crop sizing
    
- Confirmation that refinement was occurring
    
- Easier troubleshooting after encapsulation
    

### A useful software-engineering analogy

A reusable subgraph is similar to a software component.

Its main image output is the functional result.

Its diagnostic preview is a form of observability.

The implementation can remain encapsulated while exposing enough information to diagnose unexpected behaviour.

> [!tip] Subgraph design principle  
> Encapsulation should hide unnecessary complexity, not eliminate useful diagnostics.
> 
> Expose the minimum information needed to understand whether the component is doing its job.

See [[ComfyUI — Subgraph Architecture]].

## 14. Integrating Automated Detailing Into the Main Workflow

Our completed workflow supported three image-processing paths.

```
                 Base KSampler
                       │
                       ▼
                  VAE Decode
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
      Base Image   Eye Detailer  Masked Detailer
          │            │            │
          └────────────┼────────────┘
                       │
                       ▼
                 Impact Switch
                       │
                       ▼
                   Save Image
```

The switch selected one of the three available outputs.

### Mode 1 — Base Image

Returns the original generated image without additional detailing.

### Mode 2 — Eye Detailer

Runs the automated detection and refinement process.

### Mode 3 — Masked Detailer

Runs the manually controlled inpainting and compositing process.

This preserved the original generation pipeline while allowing optional refinement.

### Why place the switch after the branches?

Our final architecture used a single Impact Switch after the three processing paths.

In our tests, the switch evaluated the selected upstream path without requiring us to build a separate inverse-routing system.

An earlier attempt to combine an inverse switch with a normal switch introduced a circular graph dependency.

The simpler output-selection design avoided that problem.

See [[ComfyUI — Switches and Execution Flow]].

## 15. Diagnosing Automated Detailing Problems

|Symptom|Area to investigate|
|---|---|
|No face detected|Detector model, confidence threshold, image content|
|Wrong region selected|Bounding-box detection|
|Mask covers too much background|Segmentation or mask configuration|
|Face changes excessively|Denoise, crop, conditioning|
|Eyes remain malformed|Sampling strength, working resolution, mask coverage|
|Visible transition around refined region|Masking and compositing|
|Detailer output missing|Node execution, connections, output selection|
|Cropped preview looks unexpected|Crop sizing and diagnostic output semantics|
|Processing is unexpectedly slow|Crop resolution, detector, SAM and sampling costs|
|Different character identity|Excessive regeneration or unsuitable conditioning|

These are investigation starting points, not confirmed diagnoses.

### Recommended troubleshooting sequence

1. Verify that the detector identifies the intended face.
    
2. Inspect the mask to understand the selected region.
    
3. Inspect the cropped refinement output.
    
4. Compare the full refined image with the original.
    
5. Change one relevant parameter at a time.
    
6. Preserve the working configuration before further experimentation.
    

## 16. Key Lessons

### 1. Detection and segmentation are separate operations

YOLO identifies a target region.

SAM can provide a more precise mask.

Neither is responsible for generating the replacement image content.

### 2. The detector defines the scope of automation

A face detector does not automatically become an eye detector simply because the resulting refinement improves the eyes.

### 3. Crop size influences the balance between context and detail

A tighter crop may focus processing more effectively, but it can also remove useful surrounding information.

### 4. Denoise must be considered alongside the selected region

A setting that works well for a small manually painted eye mask may be too aggressive for a face-level detailer.

### 5. Diagnostic outputs are worth preserving

Intermediate previews help explain behaviour that would otherwise be hidden inside a reusable subgraph.

### 6. Name abstractions according to their purpose

Our `Eye Detailer` is implemented using face-level tools, but its role in the parent workflow is eye refinement.

### 7. Automated and manual detailing are complementary

Automatic detection provides convenience.

Manual masking provides precise control.

Keeping both available makes the workflow more flexible.

---

## Related Notes

- [[ComfyUI — Understanding Latent Space]]
    
- [[ComfyUI — Noise Seeds and Reproducibility]]
    
- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Switches and Execution Flow]]
    
- [[ComfyUI — Subgraph Architecture]]
    
- [[Krea 2 — Prompt Engineering]]
    
- [[Krea 2 — Practical Workflow Architecture]]
    

## Origin

Developed from practical experiments using Krea 2 Extended in ComfyUI Desktop.

The original objective was to correct malformed eyes in a café portrait without unnecessarily changing the rest of the character.

The experiment progressed through YOLO face detection, FaceDetailer refinement, SAM segmentation, crop-factor comparisons, and denoise tuning.

The successful configuration used `face_yolov9c.pt`, `sam_vit_b_01ec64.pth`, a bounding-box crop factor of 2.0, and denoise 0.30.

The resulting implementation was encapsulated as a reusable **Eye Detailer** subgraph, with a diagnostic cropped-refinement preview exposed to the parent workflow.

The architecture and conceptual distinctions are broadly applicable, while the specific settings document our tested Krea 2 configuration.