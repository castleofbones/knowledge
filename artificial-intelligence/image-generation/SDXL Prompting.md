---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- stable-diffusion-xl
    
- sdxl
    
- comfyui
    
- prompt-engineering
    
- image-generation
    

purpose: knowledge
---
Stable Diffusion XL (SDXL) is a latent diffusion image-generation architecture developed by Stability AI. SDXL Base 1.0 and checkpoints derived from it can generate images from natural-language descriptions, keyword-based prompts, or combinations of both.

Effective SDXL prompting involves understanding the relationship between:

- The checkpoint's training and aesthetic preferences.
    
- Positive and negative text conditioning.
    
- Prompt specificity, structure and emphasis.
    
- Sampling parameters, particularly CFG and denoise.
    
- The distinction between generating an image from scratch and modifying an existing image.
    

Unlike an instruction-following language model, SDXL does not reliably interpret every sentence as a strict requirement. Prompts provide conditioning that influences the generated image rather than deterministic instructions.

## 1. Natural Language and Tag-Based Prompting

SDXL supports multiple prompting styles. The most appropriate style depends on the checkpoint.

### Natural-language prompting

Natural-language prompts describe the image using ordinary descriptive language.

Example:

```
A professional macro photograph of a twenty-sided tabletop gaming die resting on a wooden table. The die is made from polished translucent ruby-red resin with white engraved numerals. Soft directional lighting illuminates the material, revealing subtle internal reflections. Shallow depth of field, realistic surface textures, detailed product photography.
```

This approach is particularly useful for specifying:

- Subjects and their relationships.
    
- Composition and camera perspective.
    
- Materials and physical appearance.
    
- Lighting and atmosphere.
    
- Photographic styles.
    

### Tag-based prompting

Tag-based prompts consist of short descriptive keywords, often separated by commas.

Example:

```
twenty-sided gaming die, d20, translucent red resin, white engraved numbers, wooden tabletop, macro photography, realistic lighting, shallow depth of field, high detail
```

Some SDXL-derived checkpoints, particularly anime-oriented models, may respond strongly to tag-based vocabulary associated with their training datasets.

However, tag-based prompting is not universally superior to natural language.

### Hybrid prompting

A hybrid prompt combines a clear natural-language description with concise aesthetic descriptors.

Example:

```
A single twenty-sided gaming die resting on a wooden tabletop, translucent ruby-red resin with engraved white numbers, precise triangular faces, realistic reflections, macro photography, shallow depth of field, soft directional lighting, intricate details
```

**Recommendation:** Start with a hybrid prompt unless the checkpoint's documentation or observed results favour a particular format.

## 2. Positive Prompt Structure

A practical SDXL prompt can be organised into five conceptual sections.

|Section|Purpose|Example|
|---|---|---|
|Subject|Defines the primary object|Twenty-sided gaming die|
|Geometry and materials|Describes construction|Triangular faces, translucent resin|
|Environment|Establishes the setting|Wooden tabletop|
|Lighting and camera|Controls presentation|Soft directional light, macro lens|
|Aesthetic|Establishes visual character|Photorealistic product photography|

A useful general structure is:

```
[Primary subject], [important physical characteristics], [environment and composition], [lighting and camera], [visual style and detail]
```

These sections do not require special syntax or a rigid order.

### Prioritising important concepts

Put the most important subject and identifying characteristics near the beginning of the prompt.

For example:

```
A single twenty-sided gaming die, a regular icosahedron with flat triangular faces and engraved white numerals, made from translucent red resin...
```

This establishes the subject before introducing less important aesthetic details.

Prompt order can influence results, but SDXL does not provide a guaranteed priority hierarchy based on word position.

## 3. Negative Prompting

Negative prompts provide conditioning associated with concepts the generation should avoid.

Example:

```
multiple dice, distorted geometry, rounded faces, malformed numerals, blurry, cartoon, illustration, watermark, signature
```

### Effective negative prompting

Negative prompts are most useful when targeting specific unwanted characteristics.

Examples include:

- Unwanted objects: `extra dice, hands, people`
    
- Geometry problems: `distorted geometry, melted edges`
    
- Style problems: `cartoon, illustration`
    
- Quality problems: `blurry, compression artifacts`
    
- Unwanted overlays: `watermark, signature`
    

### Avoid excessive negative prompts

Large negative prompts can introduce competing conditioning and may suppress desirable features.

For example, excluding all forms of irregular geometry could interfere with generating natural crystalline or organic structures.

Start with a short negative prompt and expand it only when a particular problem appears.

Negative prompts are not hard constraints and cannot guarantee the absence of unwanted details.

## 4. Prompt Weighting and Emphasis

ComfyUI's standard CLIP Text Encode nodes support emphasis syntax.

Typical examples:

```
(crystalline surface:1.2)
```

In standard ComfyUI prompt parsing, this increases the emphasis associated with the enclosed text.

Other examples:

```
(soft directional lighting:1.1)
(intricate geometric details:1.3)
```

Parentheses without an explicit numeric weight also apply emphasis in standard ComfyUI parsing.

### Weighting guidelines

|Weight|Interpretation|
|---|---|
|1.0|Normal emphasis|
|1.1|Slightly increased|
|1.2|Moderately increased|
|1.3|Stronger emphasis|
|1.5|Substantial emphasis|

These are approximate descriptions, not calibrated probabilities or guaranteed effect sizes.

### Important limitations

Prompt weighting does not guarantee that a feature will appear.

Excessive emphasis may:

- Distort subject proportions.
    
- Produce exaggerated textures.
    
- Reduce compositional coherence.
    
- Interfere with other requested features.
    

A stronger weight can also produce unexpected results because prompt concepts interact.

**Recommendation:** Use normal weighting initially. Increase emphasis only when testing a specific underrepresented feature.

Weighting syntax and behaviour can differ between applications, custom conditioning nodes and prompt parsers. The examples above assume standard ComfyUI text encoding.

## 5. Checkpoint-Specific Prompting

SDXL-derived checkpoints share an underlying architecture but may have substantially different training characteristics.

These differences can affect:

- Preferred prompt vocabulary.
    
- Response to natural language versus tags.
    
- Photorealistic or illustrative aesthetics.
    
- Colour and lighting preferences.
    
- Negative prompt sensitivity.
    
- CFG and sampling behaviour.
    

### SDXL Base 1.0

SDXL Base 1.0 provides a useful reference checkpoint for testing prompting behaviour without the additional stylistic preferences of a specialised fine-tune.

It supports natural-language descriptions and hybrid prompts.

### Fine-tuned checkpoints

A specialised checkpoint may respond differently to the same prompt because its training has altered the distribution of images it tends to generate.

For example, a photorealistic checkpoint and an anime-oriented checkpoint may interpret the same subject and lighting description differently.

**Important:** Prompting behaviour observed with one checkpoint should not automatically be generalised to another.

Checkpoint-specific recommendations should be verified against the relevant model documentation and actual generated results.

## 6. Geometry, Numbers and Spatial Relationships

SDXL can produce convincing representations of complex objects without consistently reproducing their exact geometry.

This is particularly noticeable with:

- Polyhedral dice.
    
- Mechanical assemblies.
    
- Repeating geometric structures.
    
- Text and numerals.
    
- Objects with strict spatial relationships.
    

### Polyhedral geometry

A twenty-sided gaming die is a regular icosahedron with twenty triangular faces.

A prompt such as:

```
A twenty-sided gaming die with precisely defined flat triangular faces and sharp geometric edges
```

can encourage the desired appearance.

However, SDXL may still generate:

- Incorrect face shapes.
    
- Irregular arrangements of facets.
    
- Implausible edges.
    
- Additional or missing faces.
    

### Numbers and text

SDXL does not guarantee accurate rendering of text or numbers.

Common problems include:

- Duplicated numerals.
    
- Incorrect digits.
    
- Distorted characters.
    
- Numbers crossing geometric boundaries.
    
- Decorative shapes replacing actual characters.
    

Even when the prompt explicitly requests one unique number per face, the model may fail to satisfy that requirement.

### Practical approaches

For subjects requiring accurate geometry or numbering:

1. Use a suitable reference image.
    
2. Prefer img2img with moderate or low denoise when preservation matters.
    
3. Evaluate whether geometry survives each transformation.
    
4. Use targeted inpainting or conventional image editing for corrections when necessary.
    

A prompt alone is not a reliable method of enforcing mathematically exact geometry.

## 7. Text-to-Image Versus Image-to-Image

### Text-to-image

Text-to-image generation begins with a noise latent rather than an existing photograph.

A typical ComfyUI processing path is:

```
Load Checkpoint
    |
    +-- CLIP -> Positive CLIP Text Encode
    |
    +-- CLIP -> Negative CLIP Text Encode
    |
    +-- MODEL ----------------------+
                                    |
Empty Latent Image ----------------> KSampler
                                    |
Checkpoint VAE --------------------> VAE Decode
                                    |
                                 Save Image
```

The sampler typically operates at denoise `1.0` for ordinary text-to-image generation.

This allows the model substantial freedom to construct the subject, materials and composition.

### Image-to-image

Image-to-image generation begins with an existing image encoded into latent space.

A typical ComfyUI processing path is:

```
Load Image
    |
VAE Encode
    |
KSampler (latent_image)
    |
VAE Decode
    |
Save Image
```

The checkpoint's MODEL output and positive/negative conditioning also connect to the KSampler. The VAE is used for encoding and decoding.

Img2img is particularly useful when preserving:

- Subject identity.
    
- Existing composition.
    
- Camera perspective.
    
- Overall geometry.
    
- Important colours and materials.
    

The extent of preservation depends heavily on denoise.

## 8. Understanding Denoise

Denoise controls how much noise is introduced into the latent and therefore how extensively the sampler can reinterpret the input image.

It is one of the most influential settings in img2img workflows.

### General interpretation

|Denoise|Typical behaviour|
|---|---|
|0.10–0.25|Very conservative changes|
|0.25–0.40|Surface and detail modifications|
|0.40–0.55|Moderate material and structural changes|
|0.55–0.70|Significant reinterpretation|
|0.70–0.90|Major changes to structure and composition|
|1.00|Very little effective preservation of the original image|

These are approximate ranges, not universal thresholds.

Actual behaviour depends on the checkpoint, sampler, prompt, input image and seed.

### Practical observations

In an SDXL Base 1.0 img2img workflow using a photographed d20, experiments demonstrated that lower denoise preserved the original resin material and numbers more effectively, while increasing denoise allowed greater structural reinterpretation.

The tested workflow used:

|Parameter|Value|
|---|---|
|Checkpoint|SDXL Base 1.0|
|Sampler|DPM++ 2M|
|Scheduler|Karras|
|Steps|30|
|CFG|6|
|Input|Square photograph of a d20|
|Seed|Fixed for comparisons|

The original square image was approximately 1254 × 1254 pixels; the VAE-encoded and decoded output was 1248 × 1248 pixels.

These observations establish a useful workflow for comparative experimentation, not a universal optimum for SDXL.

### Important workflow principle

When comparing denoise values, repeatedly encode the same original image.

Avoid unintentionally feeding each generated result into the next experiment, because that introduces cumulative changes and prevents a controlled comparison.

## 9. CFG Scale

Classifier-Free Guidance (CFG) controls how strongly the sampling process is guided by the difference between conditioned and unconditioned predictions.

In a standard ComfyUI workflow, the positive and negative conditioning inputs participate in this process.

### Typical effects

Lower CFG can:

- Allow more freedom in image generation.
    
- Produce less aggressive prompt adherence.
    
- Reduce some forms of overprocessed appearance.
    

Higher CFG can:

- Increase the influence of prompt conditioning.
    
- Emphasise requested characteristics.
    
- Produce exaggerated colours, textures or artifacts when pushed too far.
    

### Starting values

For conventional SDXL Base 1.0 workflows, CFG values around 5–7 can be reasonable initial experiments.

These are not universal recommendations for all SDXL-derived checkpoints.

Some fine-tuned checkpoints are designed or commonly used with substantially different guidance values.

**Recommendation:** Establish the checkpoint's expected operating range before treating a CFG value as optimal.

When comparing prompt changes, keep CFG fixed.

## 10. Sampling Steps, Samplers and Schedulers

The sampler determines how the denoising process progresses through the latent space.

The scheduler controls the noise-level sequence used during sampling.

### Sampling steps

More steps provide additional sampling iterations but do not guarantee higher image quality.

Too few steps may produce incomplete or poorly resolved results.

Beyond a certain point, additional steps may provide diminishing returns or alter the image without meaningful improvement.

For conventional SDXL workflows, approximately 20–35 steps is a reasonable initial experimental range, subject to checkpoint and sampler requirements.

### Sampler examples

**DPM++ 2M**

A commonly used deterministic sampler suitable for detailed image generation.

**Euler**

A straightforward deterministic sampler that can be useful for baseline comparisons.

**Euler Ancestral**

An ancestral sampler that introduces additional stochasticity during sampling and can produce different textures and compositions.

### Scheduler examples

**Karras**

Uses a Karras-style noise schedule and is commonly paired with DPM++ samplers.

**Normal**

Uses the scheduler's standard noise-level distribution.

Sampler and scheduler selection can materially affect the result even when the prompt and seed remain unchanged.

### Tested combination

The following combination was used successfully for SDXL Base 1.0 img2img experimentation:

```
Sampler: dpmpp_2m
Scheduler: karras
Steps: 30
CFG: 6.0
```

This is a verified working configuration, not a claim that it is optimal for every SDXL checkpoint.

## 11. Image Resolution and Aspect Ratio

SDXL Base 1.0 was developed around a native 1024-pixel image scale and supports multiple aspect ratios.

Common starting resolutions include:

|Format|Resolution|
|---|---|
|Square|1024 × 1024|
|Landscape|1216 × 832|
|Portrait|832 × 1216|
|Wider landscape|1344 × 768|
|Taller portrait|768 × 1344|

These are practical SDXL resolution choices rather than mandatory dimensions.

### Resolution considerations

- Use a resolution appropriate for the composition.
    
- Avoid stretching source photographs to a different aspect ratio.
    
- Prefer cropping or padding when preparing reference images.
    
- Higher resolution increases memory usage and processing cost.
    
- Generating far outside a checkpoint's preferred resolution range may affect composition and coherence.
    

Standard SDXL VAE encoding requires compatible spatial dimensions, normally divisible by eight.

In the observed workflow, a 1254 × 1254 input produced a 1248 × 1248 output.

## 12. Complete Prompt Examples

### Example A — Photorealistic product photography

**Positive prompt**

```
A professional macro photograph of a single twenty-sided tabletop roleplaying die, a regular icosahedron with flat triangular faces and clearly engraved white numerals, one numeral per face. The die is made from polished translucent ruby-red and emerald-green resin, with subtle internal colour swirls and realistic glossy reflections. Resting naturally on a wooden tabletop, soft diffused directional lighting, realistic material textures, shallow depth of field, detailed product photography, natural shadows.
```

**Negative prompt**

```
multiple dice, extra dice, people, hands, distorted geometry, irregular faces, melted edges, duplicated numbers, malformed numerals, cartoon, illustration, blurry, watermark, signature
```

### Example B — Fantasy artifact photography

**Positive prompt**

```
A photorealistic macro photograph of an ancient twenty-sided gaming die resting on a weathered oak table, a precisely shaped polyhedral artifact with engraved numerals and sharply defined triangular faces. Deep sapphire-blue translucent gemstone material, subtle internal mineral inclusions, polished edges worn by centuries of use, warm candlelight reflecting across the facets, dark atmospheric background, shallow depth of field, cinematic fantasy product photography, intricate material details.
```

**Negative prompt**

```
multiple dice, hands, people, distorted geometry, rounded faces, melted edges, illegible markings, cartoon, anime, flat lighting, blurry, watermark, signature
```

### Example C — Minimal hybrid prompt

**Positive prompt**

```
A single twenty-sided gaming die on a dark wooden tabletop, translucent emerald resin, crisp triangular faces, engraved white numbers, realistic reflections, soft studio lighting, macro product photography, shallow depth of field, high detail
```

**Negative prompt**

```
extra dice, distorted geometry, malformed numbers, blurry, watermark
```

These examples are starting points. They are not guaranteed to produce mathematically correct dice or numerals.

## 13. Systematic Prompt Experimentation

Prompt engineering is more effective when treated as controlled experimentation rather than repeated wholesale rewriting.

### Recommended procedure

1. Select the checkpoint and establish a working sampler configuration.
    
2. Create a concise positive prompt defining the subject and appearance.
    
3. Use a minimal negative prompt.
    
4. Fix the seed.
    
5. Generate a baseline image.
    
6. Identify one specific characteristic requiring improvement.
    
7. Change one relevant variable.
    
8. Compare the new result against the baseline.
    
9. Preserve successful configurations and outputs.
    

### Variables worth isolating

|Variable|What it tests|
|---|---|
|Prompt wording|Interpretation of subject and attributes|
|Prompt weighting|Influence of particular concepts|
|Negative prompt|Suppression of unwanted characteristics|
|CFG|Strength of conditioning guidance|
|Denoise|Degree of reference-image reinterpretation|
|Sampler|Sampling behaviour|
|Scheduler|Noise-level progression|
|Checkpoint|Learned visual preferences|

Changing several variables simultaneously can produce an improved image, but makes it harder to identify which change was responsible.

### Seed considerations

A fixed seed supports reproducibility when the remaining workflow, model and runtime conditions are unchanged.

However, identical seeds do not guarantee identical results across different samplers, checkpoints, resolutions, implementations or execution environments.

A fixed seed is a comparison tool, not a universal guarantee of determinism.

## 14. Common Prompting Problems

|Problem|Possible explanation|Targeted experiment|
|---|---|---|
|Subject not recognised|Ambiguous subject description|Use a clearer subject name|
|Incorrect geometry|Model cannot reliably enforce structure|Use img2img reference|
|Malformed numbers|Weak text-rendering capability|Preserve source markings or edit separately|
|Missing requested detail|Insufficient or competing conditioning|Simplify prompt or test emphasis|
|Excessive detail|Overemphasis or strong guidance|Reduce weighting or CFG|
|Unwanted objects|Broad or ambiguous composition|Clarify subject count and framing|
|Reference barely changes|Denoise too low|Increase denoise incrementally|
|Reference loses identity|Denoise too high|Reduce denoise|
|Unwanted illustration style|Checkpoint preference or style conditioning|Adjust aesthetic descriptors or checkpoint|

These are diagnostic hypotheses rather than guaranteed explanations.

## 15. Practical Principles

The most useful principles for SDXL prompting are:

1. **Describe the desired image, not the generation process.** SDXL responds to visual conditioning rather than procedural instructions.
    
2. **Establish the subject clearly.** Identify important geometry, materials and distinguishing features.
    
3. **Prefer clarity over length.** Longer prompts are not inherently better.
    
4. **Use checkpoint-appropriate vocabulary.** Different SDXL fine-tunes can respond differently to identical prompts.
    
5. **Treat prompt weights as experimental controls.** Stronger emphasis is not always more effective.
    
6. **Keep negative prompts targeted.** Avoid unnecessarily suppressing desired visual characteristics.
    
7. **Use reference images when exact structure matters.** Prompting alone cannot reliably preserve complex geometry or text.
    
8. **Understand the interaction between CFG and denoise.** Guidance affects conditioning, while denoise determines how extensively an input image can be reinterpreted.
    
9. **Change one variable at a time.** Controlled experiments produce reusable knowledge.
    
10. **Preserve working prompts and configurations.** Successful generations are valuable reproducible baselines.
    

## Conclusion

Effective SDXL prompting is a combination of clear visual descriptions, checkpoint awareness and controlled sampling experiments.

Natural language, tags and hybrid prompts are all valid approaches. Their effectiveness depends on the checkpoint and the subject.

For ComfyUI workflows, prompt engineering should be considered alongside CFG, sampling configuration, resolution and denoise rather than as an isolated activity.

The most reliable approach is to begin with a concise prompt, establish a reproducible baseline, evaluate actual results and make targeted adjustments supported by observed behaviour.