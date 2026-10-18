---
type: reference
status: active
created: 2026-10-18
source:
tags:
  - comfyui
  - anima
  - animij
  - lora
  - environment-plates
  - compositing
  - character-consistency
  - ai-filmmaking
---
> [!abstract] Overview
> This article documents a practical strategy for building reusable characters and environments for AI-generated filmmaking using ComfyUI.
>
> It covers canonical visual assets, environment plates, character consistency, LoRA selection, model compatibility, compositing, colour grading and the handoff to WAN 2.2 Image-to-Video.
>
> The approach developed from our experiments generating Nate Temple at Chateau Falco, where attempting to generate the character, architecture, magic and composition simultaneously repeatedly produced continuity problems.
>
> **Core principle:** Establish visual continuity before asking the video model to animate the scene.

## 1. The Problem — Too Many Responsibilities in One Generation

Our original approach attempted to generate complete character scenes using a single image-generation pass.

A typical prompt needed to describe:

- Nate's appearance and identity.
- Clothing and accessories.
- Pose and facial expression.
- Hands and magical effects.
- Chateau Falco's architecture.
- Courtyard geometry.
- Lighting and atmosphere.
- Camera angle and composition.
- Illustration style.

Each element introduced another opportunity for variation.

A generation could succeed with the character but fail with the environment.

Another could produce convincing architecture while changing Nate's clothing or appearance.

A third might create excellent magical effects but introduce unwanted symbols.

### 1.1 Examples from our experiments

We observed several recurring problems:

| Element | Intended result | Observed failure |
|---|---|---|
| Nate's shirt | Plain black crew-neck | Cross-like emblem |
| Falco | Recognisable gothic mansion | Generic country house |
| Architecture | Dark gothic revival | Cathedral-like structures |
| Environment | Broad stone courtyard | Narrow pathway or sloping causeway |
| Composition | One standing Nate | Duplicate characters |
| Magic | Small controlled starting effects | Excessive visual effects |
| Background | Gothic courtyard | Alpine village or unrelated buildings |

These were not necessarily failures of the checkpoint.

They reflected the difficulty of satisfying many interacting visual constraints in one generation.

### 1.2 Architectural change

Rather than repeatedly expanding the prompt, we decided to separate the scene into independently controlled elements.

```text
Character generation ────────┐
                             │
Environment generation ──────┼──> Compositing
                             │         |
Effects and lighting ────────┘         v
                               Approved keyframe
                                       |
                                       v
                                  WAN 2.2 I2V
                                       |
                                       v
                                 Generated shot
```

This resembles conventional film production, where actors, sets, lighting, props and visual effects are managed as separate concerns.

---

## 2. Canonical Visual Assets

A canonical asset is an approved visual reference used to establish continuity.

It represents a deliberate design decision rather than an arbitrary generation.

Examples include:

- A character's face and physical appearance.
- A standard wardrobe configuration.
- A building's architectural identity.
- A reusable environment plate.
- A recurring prop.
- A characteristic magical effect.

### 2.1 Why canonical assets matter

Without a canonical reference, each generation must independently reconstruct the intended design.

Even detailed prompts leave substantial room for interpretation.

For example, the phrase *gothic mansion* can describe many different buildings.

It does not uniquely identify Chateau Falco.

Likewise, a detailed description of Nate cannot guarantee that separate generations produce the same face.

Canonical assets reduce this ambiguity.

### 2.2 Canonical does not mean immutable

A canonical asset establishes identity, not a requirement that every shot look identical.

Falco can be shown:

- From different camera angles.
- At different times of night.
- Under different lighting conditions.
- With characters in different positions.
- With different magical effects.

The important requirement is that its defining architectural features remain recognisable.

Similarly, Nate can change pose, expression and clothing state without becoming a different character.

---

## 3. Chateau Falco as a Reusable Set

Our Scene 1 illustration established a particularly successful interpretation of Chateau Falco.

Important visual characteristics included:

- A tall central gothic architectural mass.
- A steep triangular gable.
- Narrow towers and spires.
- Pointed architectural openings.
- Dark, weathered masonry.
- A prominent central architectural feature.
- A broad stone courtyard.
- Twilight lighting.
- A dramatic graphic-novel illustration style.

These features collectively define the established visual identity of Falco.

### 3.1 Why the original image matters

The Scene 1 image was already approved as a convincing interpretation of the location.

Generating a new mansion from a text prompt would risk losing that identity.

Instead, we chose to derive a reusable environment plate from the established image.

### 3.2 Environment extraction

The process was:

1. Start with the canonical Scene 1 image.
2. Remove the unwanted chest emblem from Nate.
3. Preserve the cleaned image as **Scene 1 Master — CLEAN**.
4. Mask Nate and the associated foreground magic.
5. Reconstruct the obscured courtyard.
6. Correct the spatial interpretation of the ground.
7. Preserve the resulting environment as a candidate reusable plate.

The detailed reconstruction experiments are documented in [[ComfyUI — Inpainting and Scene Geometry]].

### 3.3 Why the extraction was difficult

The model initially interpreted the missing region as a narrow, ascending pathway toward the mansion.

This happened even when the prompt described stone paving.

The problem was primarily one of spatial interpretation.

The successful direction involved:

- A broader reconstruction mask.
- Explicit descriptions of lateral courtyard space.
- A level ground plane.
- A neutral camera perspective.
- Reduced emphasis on dramatic low-angle composition.

This reinforced an important distinction:

**An environment plate should establish coherent spatial geometry before it attempts to create dramatic cinematography.**

---

## 4. What Is an Environment Plate?

An environment plate is an image of a location intended to support compositing or further image processing.

In our workflow, a clean environment plate contains the location without the principal foreground character.

It may still include permanent or deliberately retained environmental features.

### 4.1 Environment plate versus finished background

An environment plate is not necessarily the final image used in a shot.

It may require:

- Cropping.
- Perspective adjustment.
- Exposure correction.
- Colour grading.
- Lighting changes.
- Additional props.
- Foreground elements.
- Camera-specific composition.

The plate establishes the location.

The shot determines how that location is photographed.

### 4.2 Neutral master and shot variants

A useful structure is:

```text
Falco Master Plate
        |
        +── Neutral architectural reference
        |
        +── Scene 1 background variant
        |
        +── Scene 5 background variant
        |
        +── Night exterior variant
        |
        +── Magical illumination variant
```

The master should remain available independently of its shot-specific derivatives.

### 4.3 Preserve useful lighting information

A neutral plate does not have to be uniformly illuminated.

The original Falco image already contained useful features:

- Dark stone.
- Blue twilight.
- Orange window illumination.
- Deep architectural shadows.

These contribute to the location's identity.

However, excessive brightness in the reconstructed courtyard can be corrected using downstream image processing.

There is little value in regenerating successful architecture solely to adjust exposure.

---

## 5. Character Plates

A character plate isolates the visual subject from the surrounding environment.

For our purposes, a Nate character plate should establish:

- Face and identity.
- Hair and facial hair.
- Body proportions.
- Wardrobe.
- Pose.
- Expression.
- Hand positions.
- Relevant character-specific effects.

### 5.1 Nate's established appearance

Our intended interpretation of Nate includes:

- Adult male.
- Tall, athletic build.
- Short, messy dark hair.
- Light stubble.
- Confident, slightly cocky demeanour.
- Plain black crew-neck shirt.
- Open black coat.
- Dark trousers.
- Leather boots.
- Modern clothing.
- No decorative chest emblem.

These features form a character specification.

A visual reference or character LoRA should reinforce them.

### 5.2 Why generating Nate separately helps

The character prompt no longer needs to compete with a lengthy description of Falco.

It can focus on:

- Correct identity.
- Appropriate pose.
- Wardrobe continuity.
- Facial expression.
- Hand anatomy.
- Magical effects.

The background can be handled separately.

This does not eliminate all image-generation problems, but it reduces the number of competing objectives in a single generation.

### 5.3 Character isolation

A character plate may be prepared using:

- Background removal.
- Segmentation.
- Masking.
- Inpainting.
- Manual correction.
- A combination of these techniques.

The goal is to obtain a foreground image and usable alpha mask.

For complex magical effects, the mask may need to preserve partial transparency and soft edges.

Hard-edged segmentation can damage glows, hair and translucent effects.

---

## 6. Compositing Character and Environment Plates

Compositing combines separately prepared visual elements into a single image.

Our intended pipeline is:

```text
Falco environment plate ───────────┐
                                   │
Nate character plate ──────────────┼──> Composite
                                   │         |
Character mask ────────────────────┘         |
                                             v
                                   Lighting integration
                                             |
                                             v
                                      Final keyframe
```

### 6.1 Scale and perspective

The character must be placed at a plausible position within the environment.

Important considerations include:

- Camera height.
- Ground contact.
- Relative scale.
- Perspective.
- Horizon and vanishing geometry.
- Foreground/background relationships.

A character generated from a dramatic low camera angle may not fit naturally into a neutral environment plate.

The mismatch cannot always be solved by simply resizing the character.

### 6.2 Ground contact

The character's feet or other contact points should align with the environment.

For a standing character, check:

- Foot position.
- Ground plane.
- Contact shadows.
- Apparent height.
- Perspective scale.

For a crouched character, check the relationship between knees, boots, hands and the paving.

A missing or incorrect contact shadow can make the subject appear to float.

### 6.3 Lighting integration

The foreground and background should share a plausible lighting environment.

Our Falco scenes include:

- Cool blue twilight.
- Dark architectural shadows.
- Warm orange window illumination.
- Orange-gold and blue-violet magical effects.

The character may require colour and exposure adjustment to match the environment.

Magic should also affect nearby surfaces where appropriate.

For example, an orange magical orb near Nate's hand may illuminate:

- Fingers.
- Coat sleeves.
- Nearby paving.
- Adjacent architectural surfaces.

These interactions help the composite feel like a single scene.

### 6.4 Edge treatment

A convincing composite requires more than accurate segmentation.

Check for:

- Hard cutout edges.
- Colour fringing.
- Incorrect sharpness.
- Missing glow.
- Inconsistent grain or texture.
- Mismatched illustration linework.

These defects may become more noticeable when WAN animates the composite.

---

## 7. Why Composite Before WAN?

We considered whether to generate the foreground and background separately as video elements.

For our current workflow, the preferred approach is to composite the approved still image first.

```text
Environment + Character + Effects
                |
                v
         Approved composite
                |
                v
             WAN I2V
                |
                v
          Generated video
```

### 7.1 Advantages

WAN receives a single coherent visual starting state.

This allows it to develop:

- Character movement.
- Clothing motion.
- Magical effects.
- Camera behaviour.
- Lighting interactions.
- Environmental animation.

The model is not required to invent the initial relationship between Nate and Falco.

### 7.2 Limitations

Compositing before WAN does not guarantee continuity throughout the generated sequence.

The model may still:

- Change character identity.
- Modify clothing.
- Alter architecture.
- Introduce camera movement.
- Invent additional effects.
- Change the character's expression.

However, the starting composition is better controlled.

### 7.3 Alternative production methods

Separate foreground and background video generation may eventually be useful for more advanced compositing.

That approach introduces additional challenges:

- Motion synchronisation.
- Camera matching.
- Temporal masking.
- Lighting consistency.
- Depth relationships.
- Occlusion.

It is not required for our initial shot-based pipeline.

---

## 8. What Is a LoRA?

A LoRA, or Low-Rank Adaptation, is a parameter-efficient method of adapting a pretrained model.

Rather than replacing the entire base model, a LoRA supplies learned modifications that influence model behaviour.

In image generation, LoRAs may be trained to influence:

- Character identity.
- Artistic style.
- Clothing.
- Environments.
- Architecture.
- Objects.
- Poses.
- Specific visual concepts.

### 8.1 Why LoRAs are useful

A checkpoint establishes a broad learned visual capability.

A LoRA can introduce more specialised conditioning without requiring a completely separate checkpoint.

This can reduce the need to maintain multiple large checkpoints for relatively narrow visual requirements.

### 8.2 LoRAs are not universal

LoRAs are tied to particular model architectures and training assumptions.

Compatibility must be established before loading them.

A LoRA trained for one architecture cannot be assumed to work with another.

---

## 9. Model Compatibility — Anima, Illustrious and SD1.5

Our current still-image workflow uses **Animij, an Anima-family checkpoint**.

This distinction matters when selecting LoRAs.

### 9.1 Different model families

| Model family | Compatibility consideration |
|---|---|
| Anima | Prefer LoRAs explicitly trained for the applicable Anima architecture |
| Illustrious / SDXL-derived | SDXL-family architecture; not automatically compatible with Anima |
| Stable Diffusion 1.5 | Different architecture from Anima and SDXL |
| Flux | Separate architecture and LoRA ecosystem |
| WAN | Video-generation architecture with its own compatible adapters |

Similar visual styles do not imply model compatibility.

Two checkpoints may both generate anime-style illustrations while using incompatible model architectures.

### 9.2 Compatibility checks

Before downloading a LoRA, establish:

1. The model family used for training.
2. The intended base model.
3. Whether the LoRA supports the installed checkpoint.
4. Recommended strength.
5. Trigger words.
6. Training concept and scope.
7. Example outputs.
8. Any relevant workflow or loader requirements.

Where possible, prioritise creator documentation and actual examples over the model title alone.

### 9.3 Architecture versus visual style

A Gothic Horror LoRA may sound appropriate for Falco.

However, its visual concept does not establish technical compatibility.

An SD1.5 Gothic Horror LoRA is not made compatible with Anima simply because both can generate gothic imagery.

**Compatibility first. Artistic suitability second.**

---

## 10. Case Study — GothicHorrorAI

We investigated:

[GothicHorrorAI — Civitai](https://civitai.red/models/39760/gothichorrorai)

The model was identified as an SD1.5-family LoRA.

Its intended visual style was potentially relevant to Falco.

However, its architecture did not match our Animij/Anima workflow.

### Decision

Do not insert the LoRA into the Anima workflow.

### Lesson

A visually appropriate LoRA can still be technically unsuitable.

The correct response is to find a compatible alternative rather than force an incompatible adapter into the workflow.

---

## 11. Case Study — Empty Gothic Mansion Balcony at Night

We also examined:

[Empty Gothic Mansion Balcony at Night — Anima](https://civitai.red/models/2732816/empty-gothic-mansion-balcony-at-night-anima?modelVersionId=3072530)

This was a more promising architectural match because it was explicitly associated with Anima.

However, the concept was highly specific: an empty gothic mansion balcony at night.

### 11.1 The problem with narrow concepts

A LoRA trained on a tightly focused dataset may learn more than general gothic architectural characteristics.

It may also learn:

- A recurring camera viewpoint.
- Particular architectural arrangements.
- Specific framing.
- Characteristic lighting.
- A limited range of compositions.

These learned associations can be useful when the desired image closely matches the training concept.

They can be counterproductive when the intended composition differs substantially.

### 11.2 Why this matters for Falco

Our goal is not simply to produce gothic architecture.

We need to preserve a specific established mansion and courtyard.

A LoRA strongly associated with balcony compositions might introduce:

- Balcony viewpoints.
- Unwanted architectural features.
- Changes to framing.
- Different spatial arrangements.

### Decision

Do not select a LoRA solely because its title mentions gothic architecture.

Examine the example images and determine whether the LoRA supports the required range of environments.

### Lesson

**A LoRA can learn a composition as well as a concept.**

This is especially important for narrowly trained environment LoRAs.

---

## 12. Selecting a Suitable Gothic Architecture LoRA

For Falco, the ideal LoRA would influence architectural style without forcing a specific scene layout.

Useful characteristics might include:

- Anima compatibility.
- Gothic revival architecture.
- Dark fantasy buildings.
- Weathered stonework.
- Pointed arches and windows.
- Mansions and estates.
- Multiple architectural viewpoints.
- Varied compositions.
- Exterior environments.

### 12.1 What to look for in previews

Prefer example sets showing:

- Multiple buildings.
- Different camera angles.
- Different lighting conditions.
- Varied distances.
- Consistent architectural characteristics.

Be cautious with example sets showing almost identical compositions.

### 12.2 LoRA selection is a trade-off

A highly specialised LoRA may reproduce its training concept strongly but offer limited flexibility.

A broader style LoRA may be more adaptable but less precise.

Neither is universally better.

The correct choice depends on whether the goal is:

- Reproducing a specific visual concept.
- Influencing a general style.
- Preserving a character.
- Maintaining an established environment.

For Falco, preserving location identity is more important than producing generic gothic decoration.

---

## 13. Controlled LoRA Testing

A compatible LoRA should initially be treated as an experiment.

Do not immediately incorporate it into the canonical workflow.

### 13.1 Baseline comparison

Begin with an existing successful image-generation configuration.

Preserve:

- Checkpoint.
- Seed.
- Prompt.
- Negative prompt.
- Sampler.
- Scheduler.
- Steps.
- CFG.
- Resolution.
- Mask, if inpainting.
- Denoise strength.

Then introduce the LoRA.

### 13.2 Initial strength

For an exploratory style LoRA, a low strength such as `0.25` may be a reasonable starting point.

This is a test value, not a universally recommended setting.

If the LoRA's creator documents a specific recommended range, use that information when designing the experiment.

### 13.3 Example experiment

| Test | LoRA strength | Objective |
|---|---:|---|
| A | 0.00 | Establish baseline |
| B | 0.25 | Test subtle influence |
| C | 0.40 | Test stronger influence if B is promising |

The exact values should be adapted to the LoRA.

### 13.4 Evaluation criteria

Compare:

- Architectural identity.
- Perspective.
- Ground geometry.
- Material appearance.
- Lighting.
- Composition.
- Unwanted objects.
- Style consistency.

A visually darker result is not automatically a better result.

### 13.5 Important limitation

A LoRA can affect more than its advertised concept.

A Gothic Horror LoRA might change:

- Colour palette.
- Architecture.
- Contrast.
- Composition.
- Decorative elements.

The actual behaviour must be evaluated from generated images.

---

## 14. Character LoRAs for Identity Preservation

A character LoRA serves a different purpose from an environment style LoRA.

For Nate, the primary objective would be to preserve recognisable identity across multiple still-image generations.

### 14.1 Desired consistency

A successful Nate LoRA should help maintain:

- Facial structure.
- Hair.
- Facial hair.
- General body proportions.
- Recognisable character appearance.

It should ideally permit changes in:

- Pose.
- Camera angle.
- Expression.
- Lighting.
- Environment.
- Clothing state, where required.

### 14.2 Avoid overfitting to one image

If a character LoRA learns one pose or composition too strongly, it may reproduce that pose repeatedly.

A useful character dataset should therefore represent the intended range of appearances.

The precise dataset and training strategy require separate design and testing.

### 14.3 Identity versus wardrobe

Character identity and wardrobe are related but distinct continuity concerns.

Nate's standard outfit includes a plain black shirt and open black coat.

However, a character LoRA that permanently binds identity to one outfit may become restrictive.

Where practical, character identity should remain separable from shot-specific wardrobe.

### 14.4 When not to use a character LoRA

A character LoRA is not automatically helpful for every operation.

When performing a small local repair on an existing image of Nate, the source image already contains his identity.

For example, removing the cross from his shirt required reconstructing fabric, not regenerating his face.

Adding a character LoRA to that operation could introduce unnecessary influence.

**Use a character LoRA when generating or substantially reconstructing the character, not reflexively for every local correction.**

---

## 15. LoRA Stacking

Multiple compatible LoRAs can be applied within a workflow.

Potential categories include:

- Character identity.
- Environment style.
- Clothing.
- Visual effects.
- Artistic treatment.

However, stacking is not simply a matter of adding independent effects.

### 15.1 Interaction between LoRAs

LoRAs modify model behaviour.

Their effects can interact, reinforce one another or compete.

Two LoRAs may influence overlapping model features.

The result is not guaranteed to be a simple sum of independently predictable visual changes.

### 15.2 Weight accumulation

LoRA strengths are often described as weights.

However, a weight of `0.5` on one LoRA and `0.5` on another does not necessarily mean a combined visual effect of exactly `1.0`.

The numerical scaling of parameter modifications is not equivalent to a linear measurement of perceptual influence.

### 15.3 Practical testing

Introduce LoRAs incrementally.

For example:

1. Establish a working base checkpoint.
2. Test the character LoRA alone.
3. Test the environment LoRA alone.
4. Combine them.
5. Compare the combined result with the individual tests.
6. Adjust only where the observed interaction justifies it.

### 15.4 Loader-specific behaviour

ComfyUI offers different mechanisms for loading and stacking LoRAs.

Their implementation details may differ.

Do not assume that a third-party stacking node behaves identically to a native loader without checking its actual implementation.

This is particularly relevant as ComfyUI Core and custom-node packages evolve.

---

## 16. Character, Environment and Style Separation

Our preferred architecture separates three distinct responsibilities.

### Character identity

Defines who appears in the image.

Potential tools:

- Character LoRA.
- Reference images.
- Character-specific conditioning.
- Canonical character assets.

### Environment identity

Defines where the scene takes place.

Potential tools:

- Environment plates.
- Architectural references.
- Inpainting.
- Structural conditioning.

### Shot-specific appearance

Defines how the scene is presented.

Potential tools:

- Prompting.
- Compositing.
- Lighting.
- Colour grading.
- Camera framing.
- Effects.

Conceptually:

```text
Character identity ───────────┐
                              │
Environment identity ─────────┼──> Shot construction
                              │           |
Shot-specific direction ──────┘           v
                                  Approved keyframe
```

This separation reduces the need to regenerate every aspect of a scene for every shot.

---

## 17. Managing Lighting and Magical Effects

Our Nate scenes use contrasting magical colours.

The intended effect combines:

- Orange-gold energy.
- Blue-white or blue-violet energy.
- Unstable, irregular interactions.
- Localised illumination.
- Escalating intensity.

The colours are deliberately varied.

A single uniform colour can make the magic appear too controlled or predictable.

### 17.1 Separate permanent and temporary lighting

Falco's window illumination is part of the environment.

Nate's magical light is shot-specific.

These should not necessarily be baked into the same master asset.

### 17.2 Scene 5 starting state

For Scene 5, Nate should already be standing.

His hands should contain visible but relatively restrained magical energy.

His eyes should initially appear normal.

WAN can then animate the escalation:

- Magical energy intensifies.
- Orbs become more active.
- Reflected light increases.
- His eyes begin to blaze.
- The shot ends before the action becomes difficult to control.

This gives the video model a focused performance objective.

### 17.3 Avoid overloading the source image

If the source image already contains maximum magical intensity, WAN has less room to develop a clear visual progression.

A restrained initial state can provide a more useful starting point.

---

## 18. Building a Reusable Asset Library

As the number of shots increases, asset organisation becomes important.

A conceptual library might contain:

```text
AI-Film/
│
├── Characters/
│   └── Nate/
│       ├── References/
│       ├── LoRA/
│       ├── Wardrobe/
│       └── Plates/
│
├── Environments/
│   └── Chateau-Falco/
│       ├── References/
│       ├── Master-Plates/
│       ├── Variants/
│       └── Masks/
│
├── Effects/
│   └── Wild-Magic/
│
├── Shots/
│   ├── Scene-01/
│   └── Scene-05/
│
└── Workflows/
    ├── Character-Generation/
    ├── Environment-Inpainting/
    ├── Compositing/
    └── WAN-I2V/
```

This is a proposed organisational structure, not a description of directories already created.

### 18.1 Preserve source and derived assets

For each canonical image, retain:

- Original source.
- Approved cleaned version.
- Relevant masks.
- Reconstructed plates.
- Final shot-specific composites.
- Generation metadata where available.

This allows future corrections without repeatedly regenerating the entire scene.

### 18.2 Naming conventions

Use names that distinguish asset identity from shot-specific state.

For example:

- `Falco_Master_Plate_01`
- `Falco_Scene05_Twilight`
- `Nate_Identity_Reference_01`
- `Nate_Scene05_Standing`
- `Scene05_Approved_Keyframe`

The exact naming convention can be adapted to the production's asset-management system.

---

## 19. Continuity as a Production Constraint

Visual continuity is not just an aesthetic preference.

It determines whether separate shots appear to belong to the same fictional world.

### 19.1 Character continuity

Track:

- Identity.
- Hair and facial hair.
- Wardrobe.
- Props.
- Injuries or damage.
- Current physical state.
- Magical effects.

### 19.2 Environment continuity

Track:

- Architectural identity.
- Camera orientation.
- Spatial relationships.
- Permanent props.
- Lighting conditions.
- Time of day.
- Environmental changes.

### 19.3 Shot continuity

Track:

- Starting state.
- Ending state.
- Camera position.
- Character action.
- Previous shot.
- Next shot.

A shot should not independently invent details that contradict established visual information.

### 19.4 Example — Nate's pose transition

Our first Nate clip began with him crouching.

The generated sequence eventually raised him into a standing position, but introduced unwanted changes.

Instead of using that unreliable transition, the production plan uses reaction shots.

The sequence becomes:

1. Nate prepares something reckless.
2. Alucard puts on his shades.
3. Carl giggles manically.
4. Gunter facepalms.
5. Nate is now standing as the magic escalates.

The edit communicates the transition without requiring WAN to generate it continuously.

This is a filmmaking solution to a generative-model limitation.

---

## 20. Recommended Production Pipeline

The complete proposed process is:

### Stage 1 — Establish canonical assets

Approve character and environment references.

### Stage 2 — Generate shot-specific elements

Use the appropriate model and compatible LoRAs to produce characters, poses and effects.

### Stage 3 — Prepare the environment

Select or derive the appropriate environment plate.

### Stage 4 — Composite the scene

Combine character, environment and effects.

Correct scale, perspective, shadows and lighting.

### Stage 5 — Approve the keyframe

Inspect the complete image for continuity.

Correct defects before animation.

### Stage 6 — Generate video

Pass the approved keyframe to WAN 2.2 I2V.

Use the intended motion prompt and sampling configuration.

### Stage 7 — Review the result

Inspect the entire clip.

Identify usable frames and unwanted drift.

### Stage 8 — Edit the shot

Use the successful portion in the nonlinear editor.

Avoid retaining unnecessary footage simply because the model generated it.

```text
Canonical assets
       |
       v
Shot-specific generation
       |
       v
Environment selection
       |
       v
Compositing and grading
       |
       v
Approved keyframe
       |
       v
WAN 2.2 I2V
       |
       v
Review and selection
       |
       v
Nonlinear editing
       |
       v
Completed scene
```

---

## 21. Key Lessons

> [!important] Conclusions
> **1. Separate visual responsibilities.** Character identity, environment identity and shot-specific direction should not all depend on one prompt.
>
> **2. Preserve canonical assets.** An approved character or location should become a reusable production reference.
>
> **3. Use environment plates to maintain location identity.** Falco should not be reinvented for every shot.
>
> **4. Composite before animation when practical.** WAN benefits from receiving an approved, coherent starting image.
>
> **5. Check LoRA architecture before artistic suitability.** An SD1.5 LoRA is not automatically compatible with Anima.
>
> **6. Evaluate the training concept, not just the title.** Narrow LoRAs may impose unwanted compositions.
>
> **7. Test LoRAs incrementally.** Their effects can interact and are not perceptually additive in a simple linear fashion.
>
> **8. Do not use a character LoRA unnecessarily.** Local repairs should preserve existing identity rather than regenerate it.
>
> **9. Keep neutral masters separate from shot-specific grades.** Reusable environments should support multiple lighting treatments.
>
> **10. Solve continuity before video generation.** Do not expect WAN to reconstruct the same character and location independently for every shot.

The guiding production principle remains:

**AI should improvise the performance. It shouldn't improvise the continuity.**

---

## 22. Future Development

Potential areas for further investigation:

- Training a dedicated Nate character LoRA.
- Evaluating compatible Anima gothic architecture LoRAs.
- Developing a reusable ComfyUI compositing subgraph.
- Improving foreground segmentation and edge treatment.
- Creating multiple canonical Falco camera views.
- Introducing structural conditioning for perspective control.
- Automating generation metadata capture.
- Building a formal character and environment continuity specification.

These are future development opportunities rather than established components of the current workflow.

---

## 23. Related Notes

- [[WAN 2.2 — Video Generation Fundamentals]] — Video diffusion and I2V principles.
- [[WAN 2.2 — ComfyUI Workflow Architecture]] — Dual-expert sampling, native/Turbo modes and workflow configuration.
- [[ComfyUI — Inpainting and Scene Geometry]] — Falco reconstruction, masks, perspective and conditioning.
- [[AI Filmmaking — Shot Construction and Continuity]] — Production planning, visual continuity and editing.

---

## 24. References

### Technical resources

- [ComfyUI documentation](https://docs.comfy.org/)
- [ComfyUI source repository](https://github.com/comfyanonymous/ComfyUI)
- [WAN 2.2 official repository](https://github.com/Wan-Video/Wan2.2)

### LoRA examples

- [GothicHorrorAI — SD1.5 LoRA](https://civitai.red/models/39760/gothichorrorai)
- [Empty Gothic Mansion Balcony at Night — Anima](https://civitai.red/models/2732816/empty-gothic-mansion-balcony-at-night-anima?modelVersionId=3072530)

### Internal project assets and experiments

- Nate Scene 1 original illustration.
- Scene 1 Master — CLEAN.
- Falco courtyard reconstruction experiments.
- Scene 5 standing Nate candidate.
- WAN 2.2 native and Turbo video experiments.

The project observations documented here are based on our own ComfyUI experiments. Proposed future workflows and asset structures are identified separately from implemented components.