---

type: reference  
status: active  
created: 2026-10-18  
source: ChatGPT — ComfyUI / Image Generation project  
tags:

- comfyui
    
- inpainting
    
- anima
    
- animij
    
- scene-geometry
    
- perspective
    
- masking
    
- image-generation
    
- ai-filmmaking
    

purpose: knowledge
---
> [!abstract] Overview  
> Inpainting allows selected regions of an existing image to be regenerated while preserving surrounding content.
> 
> This article covers practical inpainting with ComfyUI, including masking, conditioning, denoise strength, scene geometry, perspective and iterative correction.
> 
> It also documents our experiments reconstructing the courtyard of **Chateau Falco** from an established character illustration.
> 
> The central lesson is that successful inpainting requires more than describing replacement objects. The model must also infer the correct spatial relationships between the surviving image and the missing region.

## 1. What Is Inpainting?

Inpainting is an image-generation technique in which a model reconstructs selected areas of an existing image.

Rather than generating an entirely new composition, the model uses the surviving image and text conditioning to infer plausible replacement content.

Typical applications include:

- Removing unwanted objects or characters.
    
- Repairing clothing and facial details.
    
- Replacing backgrounds.
    
- Correcting visual artefacts.
    
- Reconstructing obscured architecture.
    
- Extending or modifying environmental features.
    
- Preparing clean images for compositing.
    

Inpainting is particularly useful when most of an image is already correct.

### 1.1 Inpainting versus full regeneration

Full image generation asks the model to create an entire scene.

Inpainting asks the model to modify a selected region while preserving the rest.

This distinction matters when an image already contains valuable details such as:

- Established character identity.
    
- Correct architecture.
    
- Successful lighting.
    
- Desired composition.
    
- Consistent illustration style.
    

Regenerating everything introduces opportunities to lose those details.

**Prefer a targeted correction when the existing image is already substantially correct.**

---

## 2. The Basic ComfyUI Inpainting Workflow

Our experiments used an Anima-family checkpoint, Animij, within ComfyUI Desktop.

The workflow followed this general structure:

```
Load Image
    |
    +----> Image -----------------------+
    |                                   |
    +----> Mask                         |
              |                         |
              v                         |
       VAE Encode (for Inpainting) <----+
              |
              v
           KSampler
              |
              v
          VAE Decode
              |
              v
          Preview Image
              |
              v
           Save Image
```

The KSampler also receives:

- The appropriate model.
    
- Positive conditioning.
    
- Negative conditioning.
    
- Sampler and scheduler configuration.
    
- Seed.
    
- Steps and CFG.
    

The diagram is conceptual. In the actual graph, the image and mask connect to the appropriate inputs of `VAE Encode (for Inpainting)`, and the model and conditioning follow their own paths.

### 2.1 Mask creation

For our workflow, masks were created using ComfyUI's image mask editor.

The general procedure was:

1. Load the image using `Load Image`.
    
2. Open the mask editor from the image node.
    
3. Paint the region requiring replacement.
    
4. Preserve the surrounding image.
    
5. Return to the workflow and execute the inpainting branch.
    

A mask determines which region is eligible for reconstruction.

It does not, by itself, specify what should replace the removed content.

That information must be inferred from conditioning and the surrounding image.

---

## 3. Understanding Masks

A mask is more than a selection of unwanted pixels.

Its shape, extent and relationship to the surrounding image affect the reconstruction problem presented to the model.

### 3.1 Tight masks

A tight mask follows the outline of the object being removed.

This can be appropriate for small, localised corrections.

Examples include:

- A clothing emblem.
    
- A minor surface defect.
    
- A small unwanted accessory.
    

However, a tight mask may be problematic when removing a large subject.

The model can still see surrounding evidence of the removed object, such as shadows, reflected light or compositional structure.

### 3.2 Broad masks

A broad mask includes the subject and related visual effects.

This can be more appropriate when removing:

- A person occupying much of the foreground.
    
- Large magical effects.
    
- Strong cast shadows.
    
- Objects that obscure architectural geometry.
    

A broad mask gives the model more freedom to reconstruct a coherent region.

The trade-off is that more original image information must be regenerated.

### 3.3 Mask expansion

Our workflow used the `grow_mask_by` setting on `VAE Encode (for Inpainting)`.

This expands the masked region by a specified amount.

In our experiments:

- Small clothing correction: approximately `6`.
    
- Larger subject removal: approximately `16`.
    

These are recorded experimental settings, not universal values.

The appropriate expansion depends on image resolution, mask shape and the amount of surrounding context.

### 3.4 Why mask shape matters

Consider two masks covering approximately the same central subject.

**Tight subject-shaped mask:**

```
+-------------------------+
|                         |
|         FALCO           |
|                         |
|           ██            |
|          ████           |
|         ██████          |
|          ████           |
|         ██  ██          |
|                         |
+-------------------------+
```

**Broad courtyard reconstruction mask:**

```
+-------------------------+
|                         |
|         FALCO           |
|                         |
|-------------------------|
|  █████████████████████  |
|  █████████████████████  |
|  █████████████████████  |
|  █████████████████████  |
|  █████████████████████  |
+-------------------------+
```

The first asks the model to fill a character-shaped gap.

The second asks it to reconstruct a larger continuous environmental surface.

Both may contain the same unwanted character, but they present different reconstruction problems.

> [!important] Experimental lesson  
> The mask controls the spatial scope of the reconstruction.
> 
> Changing its shape can change the geometric interpretation of the missing region, even when the prompt and model remain similar.
> 
> The mask does not explicitly encode geometry, however. The result still depends on the surviving pixels, conditioning and model behaviour.

---

## 4. Denoise Strength

Denoise strength controls how extensively the sampling process modifies the encoded image information.

Broadly:

- Lower denoise tends to preserve more of the original image.
    
- Higher denoise permits greater reconstruction.
    
- Excessive denoise may produce unrelated or unexpected content.
    

The effect also depends on the inpainting implementation, sampler, model and conditioning.

### Recorded experimental values

|Task|Denoise|Observed purpose|
|---|---|---|
|Remove chest emblem|~0.30|Preserve fabric and clothing structure|
|Initial character removal|~0.65|Reconstruct obscured environment|
|Secondary courtyard repair|~0.40–0.50|Attempt to correct invented paving|

These values should be treated as starting points for similar experiments, not fixed recommendations.

### 4.1 Small correction versus large reconstruction

A clothing emblem requires relatively little new scene information.

The model mainly needs to reconstruct nearby fabric.

Removing an entire foreground character is fundamentally different.

The model must infer:

- Previously hidden paving.
    
- Architectural surfaces.
    
- Perspective.
    
- Shadows.
    
- Spatial relationships.
    
- Lighting continuity.
    

A larger reconstruction may require greater denoising freedom.

However, more freedom also increases the possibility of invented objects and geometry.

---

## 5. Conditioning for Inpainting

Text conditioning describes the desired replacement content.

A useful inpainting prompt should focus on what belongs inside the masked region.

### 5.1 Positive conditioning

Positive conditioning should describe the intended result.

For example, when repairing a black shirt:

```
plain black crew-neck shirt, solid black fabric, natural fabric folds, matte black fabric
```

This describes the replacement material without unnecessarily reintroducing the unwanted object.

### 5.2 Negative conditioning

Negative conditioning can identify unwanted concepts.

For the shirt correction:

```
cross, crucifix, symbol, emblem, logo, print, graphic, necklace, pendant
```

Negative conditioning is not a guaranteed exclusion mechanism.

It influences generation rather than enforcing a hard constraint.

### 5.3 Avoiding negation in positive prompts

During our Animij experiments, positive phrases such as:

`no cross, no crucifix`

were unreliable.

One plausible contributing factor is that the text encoder still receives the concepts `cross` and `crucifix`, despite the surrounding negation.

For our Danbooru-influenced prompting approach, it proved more practical to describe the desired appearance positively and reserve unwanted concepts for negative conditioning.

**Describe what should exist rather than repeatedly naming what should not exist.**

---

## 6. Case Study — Removing Nate's Chest Emblem

### 6.1 Starting problem

Our canonical Nate illustration showed the character crouching in Falco's courtyard.

The image had several successful elements:

- Strong character appearance.
    
- Convincing magic.
    
- Appropriate gothic atmosphere.
    
- Useful composition.
    
- Recognisable architecture.
    

Unfortunately, Animij repeatedly generated a cross-like emblem on Nate's otherwise plain black shirt.

This conflicted with the established character design.

The emblem became an unwanted recurring feature across generations.

### 6.2 Why not regenerate the image?

The rest of the image was already successful.

Full regeneration risked changing:

- Nate's face.
    
- His pose.
    
- The magic.
    
- Falco's architecture.
    
- Lighting.
    
- Composition.
    

A local repair was more appropriate.

### 6.3 Inpainting approach

We masked the chest emblem with a small surrounding margin.

The inpainting prompt focused on plain black fabric.

**Positive prompt:**

```
plain black crew-neck shirt, solid black fabric, natural fabric folds, matte black fabric
```

**Negative prompt:**

```
cross, crucifix, symbol, emblem, logo, print, graphic, necklace, pendant
```

Recorded settings included:

- `grow_mask_by`: approximately `6`.
    
- Denoise: approximately `0.30`.
    
- Existing sampler, scheduler, steps and CFG preserved.
    

### 6.4 Result

The unwanted emblem disappeared.

The reconstructed shirt retained plausible folds and shading.

A small warm highlight remained, but it did not resemble a symbol.

The corrected image was accepted as:

**Scene 1 Master — CLEAN**

### 6.5 Lesson

For small, localised defects, a narrow mask and relatively conservative denoise can preserve the successful parts of an image.

The model did not need to regenerate Nate.

It only needed to reconstruct a small area of fabric.

---

## 7. Case Study — Removing Nate to Recover Falco

### 7.1 Objective

The next task was to create a reusable environment plate from the cleaned Scene 1 image.

The intention was to remove:

- Nate.
    
- His foreground magical effects.
    
- Visual features associated with his presence.
    

The surviving architecture would establish the canonical appearance of Chateau Falco.

This would allow the same environment to be reused in later shots.

See [[ComfyUI — Environment Plates and LoRA Strategy]].

### 7.2 Why extract the environment?

Repeatedly asking Animij to generate Nate and Falco together had produced inconsistent results.

The model was simultaneously responsible for:

- Character identity.
    
- Pose.
    
- Wardrobe.
    
- Hands.
    
- Magic.
    
- Architecture.
    
- Camera angle.
    
- Lighting.
    
- Composition.
    

The resulting images often succeeded in some areas while failing in others.

Separating character and environment generation would reduce the number of simultaneous constraints.

The intended pipeline became:

```
Canonical Falco image
         |
         v
Remove Nate and foreground magic
         |
         v
Falco environment plate
         |
         +------------------+
         |                  |
         v                  v
Scene 1 composition    Scene 5 composition
```

### 7.3 Initial mask

The first attempt masked Nate and nearby foreground magic.

The original Falco architecture was largely preserved.

### 7.4 Initial prompt

The first reconstruction prompt emphasised gothic architecture, stone paving and twilight lighting.

**Positive prompt:**

```
gothic revival mansion exterior, dark weathered stone architecture, tall pointed windows, stone columns, pointed arches, dark entrance, grand stone terrace, weathered stone paving, architectural details, deep shadows, blue twilight illumination, orange and blue reflected magical light, graphic novel, comic book illustration, detailed linework, dramatic ink shading, rich saturated colors
```

**Negative prompt:**

```
1boy, man, male, person, people, human, character, face, body, hands, legs, boots, silhouette, cross, crucifix, religious symbol, text, logo
```

Recorded denoise was approximately `0.65`.

---

## 8. First Result — Successful Removal, Invented Environment

The initial large inpainting operation was surprisingly successful.

### What worked

- Nate was removed.
    
- The upper portion of Falco remained recognisable.
    
- The model reconstructed plausible architecture.
    
- Stone paving appeared where the character had stood.
    
- The image retained much of its original visual style.
    

### What failed

The model introduced a strong central arrangement of glowing orange diamond-like shapes.

The courtyard began resembling a magical runway leading toward the mansion entrance.

Some reconstructed architectural features also became more cathedral-like.

### Likely contributing factors

The surviving scene contained:

- Orange magical illumination.
    
- Perspective lines converging toward the building.
    
- A central architectural entrance.
    
- Strong compositional symmetry.
    

The prompt also included references to orange reflected magical light.

A plausible interpretation is that the model incorporated those cues into the reconstructed paving.

This remains an interpretation of the observed behaviour, not a confirmed internal mechanism.

---

## 9. Failed Approach — Repairing the Repair

Our next idea was to remove the invented glowing runway using a second inpainting pass.

### Proposed correction

Mask the unwanted orange features and regenerate ordinary dark stone paving.

The new prompt emphasised unremarkable courtyard surfaces.

Denoise was reduced to approximately `0.40–0.50`.

### Result

The model generated increasingly dramatic and inappropriate features, including:

- A large magical crater.
    
- Broken stone.
    
- Impact-like formations.
    
- Floating or displaced rocks.
    

Changing the seed did not reliably solve the problem.

### Interpretation

The second pass inherited visual context from an already incorrect reconstruction.

The surviving scene continued to suggest a significant central feature.

The new mask removed some unwanted pixels but did not necessarily remove the broader geometric cues supporting them.

### Lesson

**Repeatedly repairing an incorrect reconstruction can reinforce the wrong scene interpretation.**

When the underlying geometry is wrong, returning to the last known-good source may be more productive than repeatedly modifying a flawed intermediate image.

This led us back to **Scene 1 Master — CLEAN**.

---

## 10. Second Approach — Broader Foreground Removal

We returned to the original cleaned image.

The new mask covered:

- Nate.
    
- His immediate foreground magic.
    
- More of the orange illuminated ground.
    
- A broader portion of the central paving.
    

The intention was to remove visual evidence that could encourage the model to reconstruct a magical runway.

### Revised prompt

**Positive prompt:**

```
empty stone terrace, weathered dark stone paving, continuous stone floor, irregular rectangular paving stones, natural perspective, cracked old stone, dark gothic mansion architecture in background, blue twilight illumination, subtle orange and blue reflected light, graphic novel, comic book illustration, detailed linework, dramatic ink shading
```

**Negative prompt:**

```
1boy, man, person, human, character, statue, figure, silhouette, magic circle, rune, sigil, symbol, glowing floor, glowing paving, floor light, altar, pedestal, monument, hole, crater, pit, rubble, floating rocks
```

### Result

The model continued to construct a central pathway or causeway.

Orange decorative features remained a recurring problem.

### Important observation

The problem was no longer simply unwanted magical detail.

The model was reconstructing the **wrong spatial layout**.

---

## 11. Identifying the Real Problem — Scene Geometry

The key observation was that the generated ground appeared to lead directly toward Falco's entrance.

The model seemed to interpret the scene as:

```
Camera
   |
   v
Long paved pathway
   |
   v
Mansion entrance
```

But the intended geometry was:

```
Camera
   |
   v
Foreground terrace
   |
   v
Broad open courtyard
   |
   v
Mansion facade
```

These are substantially different scenes.

### 11.1 Why the distinction matters

A pathway is a narrow directional feature.

A courtyard is an expansive area with significant lateral space.

Both may contain stone paving.

Both may recede toward a building.

However, their geometry and visual implications differ.

The earlier prompts described paving materials but did not sufficiently establish the spatial arrangement.

### 11.2 Material description versus spatial description

Examples of material-focused terms:

- Weathered stone.
    
- Cracked paving.
    
- Dark masonry.
    
- Irregular slabs.
    

Examples of spatial terms:

- Broad open courtyard.
    
- Wide lateral space.
    
- Large empty area.
    
- Mansion at the far side.
    
- Horizontal paving rows.
    
- Building separated from the foreground by open ground.
    

Material terms describe what a surface looks like.

Spatial terms describe how surfaces and objects relate to one another.

**When geometry is wrong, adding more material adjectives is unlikely to solve the fundamental problem.**

---

## 12. Successful Direction — Reconstruct the Courtyard as a Whole

We changed both the mask strategy and the prompt.

Rather than asking the model to fill the space previously occupied by Nate, we asked it to reconstruct the lower courtyard as a coherent surface.

### 12.1 Mask strategy

The mask became a broad rectangular region covering much of the lower and middle image.

The goal was to remove the narrow central composition that encouraged pathway generation.

We preserved as much useful original architecture as practical.

Where possible, the surviving image provided boundaries around the reconstructed ground.

Conceptually:

```
+--------------------------------+
|                                |
|        FALCO ARCHITECTURE      |
|                                |
|--------------------------------|
|                                |
| ██████████████████████████████ |
| ██████████████████████████████ |
| ██████████████████████████████ |
| ██████████████████████████████ |
| ██████████████████████████████ |
|                                |
+--------------------------------+
```

### 12.2 Complete revised positive prompt

```
masterpiece, best quality, very aesthetic, absurdres,

empty exterior courtyard,
large broad open courtyard,
wide rectangular stone courtyard,
expansive open space,

foreground weathered stone terrace,
large irregular stone slabs in foreground,
broad horizontal paving pattern,
wide stone paving extending left and right,

open courtyard in middle distance,
large empty courtyard between camera and mansion,
broad flat open area,
wide lateral space,
horizontal rows of paving stones,
subtle perspective toward background,

gothic mansion at far side of courtyard,
mansion separated from foreground by large open courtyard,
dark gothic mansion facade in distance,
grand mansion entrance facing courtyard,
mansion occupying upper background,
mansion set back from camera,

blue twilight illumination,
subtle orange and blue reflected light,
deep shadows,

graphic novel,
comic book illustration,
detailed linework,
dramatic ink shading,
rich saturated colors
```

### 12.3 Complete revised negative prompt

```
1boy, man, person, human, character,
statue, figure, silhouette,

path, pathway, walkway, causeway,
road, street, corridor,
narrow paving, narrow passage,
paved path leading to door,
central path, central walkway,

leading line, converging path,
runway, track,

magic circle, rune, sigil, symbol,
crystal, gemstone, glowing crystal,
altar, pedestal, monument,
glowing floor, glowing paving,
floor light, floor decoration,

hole, crater, pit,
rubble, floating rocks
```

### 12.4 Result

The broader reconstruction began producing substantially more convincing courtyard geometry.

The image no longer read primarily as a narrow central causeway.

However, a second spatial problem remained.

---

## 13. Perspective Error — The Courtyard Was Still Uphill

Although the courtyard became broader, its perspective still suggested significant vertical elevation.

The ground appeared to rise toward Falco.

The mansion seemed to occupy higher ground than the camera.

This was not the intended scene.

### 13.1 Camera angle versus ground slope

These are different geometric properties.

**Low camera angle:** The camera looks upward relative to the scene.

**Sloping ground:** The physical ground surface rises with distance.

A low camera position does not require the ground to slope upward.

Likewise, a level courtyard can be photographed from many camera heights.

Our original Nate composition used a dramatic perspective.

That was useful for the character illustration but less desirable when extracting a reusable architectural plate.

### 13.2 Revised objective

The environment plate should use a relatively neutral architectural viewpoint.

The ground should remain level.

Dramatic character framing can be established separately during compositing and shot construction.

### 13.3 Complete level-courtyard positive prompt

```
masterpiece, best quality, very aesthetic, absurdres,

empty exterior courtyard,
large broad open courtyard,
wide rectangular stone courtyard,
expansive open space,

perfectly level courtyard,
flat horizontal ground plane,
level stone surface,
flat stone paving,
courtyard at constant elevation,
mansion and foreground standing on the same level ground plane,

foreground weathered stone terrace,
large irregular stone slabs in foreground,
broad horizontal paving pattern,
wide stone paving extending left and right,

large empty courtyard in middle distance,
broad open space between camera and mansion,
wide lateral space,
horizontal rows of paving stones,
evenly receding paving,
natural perspective across flat ground,

gothic mansion at far side of courtyard,
mansion separated from foreground by large open courtyard,
dark gothic mansion facade in distance,
grand mansion entrance facing courtyard,
mansion occupying upper background,
mansion set back horizontally from camera,

camera positioned approximately at standing eye level,
camera looking horizontally across courtyard,
level camera,
straight horizon,
vertical architectural lines,
balanced architectural perspective,

blue twilight illumination,
subtle orange and blue reflected light,
deep shadows,

graphic novel,
comic book illustration,
detailed linework,
dramatic ink shading,
rich saturated colors
```

### 13.4 Complete level-courtyard negative prompt

```
1boy, man, person, human, character,
statue, figure, silhouette,

sloped ground, uphill, hill, hillside,
incline, inclined ground, ramp,
rising path, ascending path,
stairs, steps, staircase,
elevated mansion, mansion on hill,

low angle, extreme low angle,
worm's-eye view,
tilted camera, dutch angle,
distorted perspective,
exaggerated perspective,
steep perspective,

path, pathway, walkway, causeway,
road, street, corridor,
narrow paving, narrow passage,
central path, central walkway,
runway, track,

magic circle, rune, sigil, symbol,
crystal, gemstone, glowing crystal,
altar, pedestal, monument,
glowing floor, glowing paving,
floor light, floor decoration,

hole, crater, pit,
rubble, floating rocks
```

### 13.5 Result

Subsequent generations showed substantially improved perspective.

The courtyard began reading as a broad, level area in front of Falco.

The mansion remained recognisable.

The strong central runway interpretation was largely eliminated.

The resulting image was significantly closer to a reusable environment plate.

### Remaining issue

The generated image was somewhat brighter than desired.

This was primarily a tonal issue rather than a geometry problem.

At this stage, we decided not to disturb the successful reconstruction simply to change its exposure.

---

## 14. Separating Geometry from Colour Grading

Once the environment geometry is acceptable, tonal correction can be handled downstream.

This is preferable to asking the diffusion model to reconstruct the scene again solely to make it darker.

### 14.1 Proposed workflow

```
Inpainting result
       |
       v
   VAE Decode
       |
       v
Exposure / Levels / Curves
       |
       v
Corrected environment plate
       |
       v
Save Image
```

The exact grading node depends on the installed ComfyUI node packages.

The intended correction is conceptual rather than tied to an unverified custom node.

### 14.2 Why not simply reduce brightness?

The Falco image already contains useful dark and luminous elements.

These include:

- Dark masonry.
    
- Blue twilight sky.
    
- Orange illuminated windows.
    
- Deep architectural shadows.
    

A uniform brightness reduction may damage those relationships.

A more useful grade would reduce excessive midtone brightness while preserving:

- Shadow separation.
    
- Architectural detail.
    
- Window illumination.
    
- Colour contrast.
    

Levels, curves or gamma-based adjustments may offer more control than a simple brightness multiplier.

### 14.3 Preserve a neutral master

A reusable environment plate should not necessarily contain the final grade for every shot.

Different scenes may require different lighting treatments.

For example:

```
Falco Master Plate
        |
        +--> Scene 1: subdued twilight
        |
        +--> Scene 5: stronger magical illumination
        |
        +--> Later scene: darker night
        |
        +--> Establishing shot: neutral grade
```

This separates environment identity from shot-specific lighting.

---

## 15. Why Repeated Prompt Changes Were Not Enough

Several early attempts concentrated on refining the text prompt.

We tried increasingly explicit descriptions of:

- Stone paving.
    
- Architectural detail.
    
- Empty ground.
    
- Missing characters.
    
- Unwanted magical features.
    

Some changes improved local details.

However, the main problem persisted because the reconstructed area still implied an incorrect spatial layout.

The important improvement came when we changed the reconstruction task itself.

Instead of:

**Replace Nate with whatever lies behind him.**

We moved toward:

**Reconstruct the broad courtyard as one continuous, level architectural space.**

The revised mask and conditioning supported the same spatial objective.

This is a useful general principle for image generation:

> [!tip] Diagnose the category of failure  
> A texture problem may require better material conditioning.
> 
> A lighting problem may require grading.
> 
> A composition problem may require framing changes.
> 
> A geometry problem may require structural conditioning or a different mask.
> 
> Treating every failure as a prompt-wording problem wastes time.

---

## 16. Inpainting and Structural Conditioning

Our experiments relied primarily on image context, masks and text conditioning.

For more demanding architectural reconstruction, stronger structural guidance may be useful.

Potential approaches include:

- Depth conditioning.
    
- Line-art conditioning.
    
- Edge-based guidance.
    
- Perspective guides.
    
- Rough painted geometry.
    
- Reference-image conditioning.
    
- Manual compositing.
    

These techniques are not interchangeable.

Their availability and compatibility depend on the model architecture and installed ComfyUI nodes.

### 16.1 Depth conditioning

A depth map can provide information about relative scene distance.

This may help preserve broad spatial relationships.

However, depth does not directly guarantee a level ground plane.

A geometrically incorrect depth map can reinforce the wrong perspective.

### 16.2 Line-art or edge conditioning

Line-art guidance can preserve architectural outlines and paving boundaries.

This may be useful when the desired geometry is already known.

However, preserving incorrect perspective lines can also preserve the original problem.

### 16.3 Rough geometry guides

A manually prepared image showing the desired ground plane and major boundaries can provide more explicit spatial information than text alone.

The guide does not need to be visually polished.

Its purpose is to establish layout.

### 16.4 When to introduce structural guidance

Consider structural conditioning when:

- Repeated prompts produce the same geometry error.
    
- Seed changes do not materially improve the layout.
    
- The mask cannot remove the problematic contextual cues.
    
- Exact perspective is important.
    
- The reconstruction must match an established environment.
    

Do not introduce additional control mechanisms when the existing workflow already produces acceptable results.

---

## 17. Inpainting as Part of a Compositing Pipeline

The Falco experiment began as an object-removal task.

It evolved into a method for producing reusable environment assets.

The intended production pipeline is:

```
Canonical scene image
        |
        v
Remove character and effects
        |
        v
Reconstruct environment
        |
        v
Correct perspective
        |
        v
Apply neutral grade
        |
        v
Falco environment plate
        |
        +---------------------+
        |                     |
        v                     v
Character composite      Alternate shot
        |
        v
Approved keyframe
        |
        v
WAN 2.2 Image-to-Video
        |
        v
Generated video shot
```

The purpose is not merely to remove an unwanted character.

It is to separate the environment from the performance so that both can be controlled independently.

This is developed further in [[ComfyUI — Environment Plates and LoRA Strategy]].

---

## 18. Experimental History

|Experiment|Change|Result|Lesson|
|---|---|---|---|
|Chest emblem removal|Small mask, low denoise|Successful|Local repair preserves good imagery|
|Initial Nate removal|Character-shaped mask, higher denoise|Nate removed, runway invented|Removing a subject does not guarantee correct geometry|
|Runway repair|Secondary mask, lower denoise|Crater and magical debris|Repairing an incorrect reconstruction can compound errors|
|Larger foreground removal|Broader mask|Central pathway persisted|Material conditioning alone was insufficient|
|Broad courtyard reconstruction|Wide rectangular mask, spatial prompt|Improved courtyard width|Mask scope and spatial conditioning matter|
|Level-ground conditioning|Explicit flat ground and camera geometry|Improved perspective|Separate camera angle from ground slope|
|Tonal correction planning|Move exposure changes downstream|Avoids unnecessary regeneration|Grade successful geometry rather than regenerating it|

### Interpretation

The experiments support the practical value of matching mask design and conditioning to the intended spatial reconstruction.

They do not isolate a single causal variable across every attempt.

Multiple factors changed during the development process, including mask extent, conditioning and reconstruction context.

---

## 19. Practical Inpainting Procedure

When approaching a new inpainting task:

### Step 1 — Identify the actual defect

Determine whether the problem is:

- Local visual detail.
    
- Object removal.
    
- Lighting.
    
- Composition.
    
- Geometry.
    
- Style consistency.
    

### Step 2 — Preserve the known-good source

Keep an unchanged copy of the best existing image.

Do not overwrite the original with experimental repairs.

### Step 3 — Design the mask for the task

For small corrections, use a focused mask.

For large environmental reconstruction, consider a broader region that allows coherent geometry.

Include related shadows and effects where necessary.

### Step 4 — Describe the desired replacement

Use positive conditioning to establish what belongs in the region.

Use negative conditioning to discourage unwanted concepts.

Avoid excessively redundant prompts.

### Step 5 — Choose an appropriate denoise level

Begin with a value suited to the amount of reconstruction required.

Change it only when the observed result justifies doing so.

### Step 6 — Evaluate structure before detail

Check:

- Perspective.
    
- Scale.
    
- Ground plane.
    
- Architectural continuity.
    
- Spatial relationships.
    

Do not spend time polishing textures if the geometry is wrong.

### Step 7 — Change one relevant variable

Where practical, preserve the seed and other settings.

Compare the resulting changes.

### Step 8 — Return to the original if necessary

If repeated repairs compound the problem, restart from the known-good source with a revised reconstruction strategy.

### Step 9 — Separate finishing operations

Once geometry and content are correct, use downstream image processing for exposure, colour and tonal adjustments.

### Step 10 — Save reusable assets

Preserve successful environment plates and document the settings used to produce them.

---

## 20. Key Lessons

> [!important] Conclusions  
> **1. Inpainting is a reconstruction problem, not simply an eraser.** The model must infer what belongs behind removed content.
> 
> **2. Mask shape and extent matter.** A character-shaped hole and a broad courtyard region present different reconstruction tasks.
> 
> **3. The surrounding image can reinforce unwanted geometry.** Converging lines, illumination and surviving features influence the result.
> 
> **4. Material descriptions do not define spatial layout.** A prompt describing stone paving does not necessarily establish a broad, level courtyard.
> 
> **5. Perspective and ground slope are different properties.** A dramatic camera angle does not require an inclined surface.
> 
> **6. Repeated repair passes can compound errors.** Return to the last known-good source when the reconstructed scene becomes structurally incorrect.
> 
> **7. Geometry should be solved before colour grading.** Do not regenerate a successful scene solely to change brightness.
> 
> **8. Preserve canonical assets.** A successful environment reconstruction can become a reusable set rather than a one-off image.
> 
> **9. Use stronger structural conditioning when text and masks are insufficient.** But do not add unnecessary complexity to a working process.
> 
> **10. Evaluate results empirically.** Prompt wording, denoise and seed are hypotheses to test, not guarantees of particular outcomes.

---

## 21. Related Notes

- [[WAN 2.2 — Video Generation Fundamentals]] — Video diffusion, I2V conditioning and temporal consistency.
    
- [[WAN 2.2 — ComfyUI Workflow Architecture]] — Dual-expert sampling and practical video generation.
    
- [[ComfyUI — Environment Plates and LoRA Strategy]] — Canonical environments, character separation and compositing.
    
- [[AI Filmmaking — Shot Construction and Continuity]] — Reusable sets, approved keyframes and editing.
    

---

## 22. References

### Technical resources

- [ComfyUI documentation](https://docs.comfy.org/)
    
- [ComfyUI source repository](https://github.com/comfyanonymous/ComfyUI)
    

### Internal experiments

- Nate Scene 1 — Original character illustration.
    
- Nate Scene 1 Master — CLEAN.
    
- Falco courtyard reconstruction experiments.
    
- Broad-mask courtyard reconstruction.
    
- Level-ground perspective correction.
    
- Planned environment-plate colour grading.
    

The technical observations and recorded settings in this article originate from our local ComfyUI experiments. They are not presented as universal model behaviour.