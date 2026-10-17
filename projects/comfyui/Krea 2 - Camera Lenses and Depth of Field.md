---

type: reference  
status: active  
created: 2026-10-17  
source: ""  
tags:

- comfyui
    
- krea-2
    
- image-generation
    
- prompt-engineering
    
- depth-of-field
    
- photography
    

---
# Camera Lenses and Depth of Field

## Overview

Krea 2 can be guided using conventional photographic terminology, including focal length, aperture, focus distance and depth of field.

However, specifying photographic parameters does not guarantee that the model will reproduce their precise optical effects.

**The central prompting principle is to describe both the photographic technique and the intended visual result.**

For example, specifying `85mm lens, f/2.8` provides photographic context, but explicitly describing which elements should remain sharp and which should become defocused gives the model a clearer target.

This approach is particularly useful when generating realistic photographic scenes in ComfyUI.

## 1. Photographic Parameters vs Visual Intent

In conventional photography, focal length, aperture, subject distance, focus distance and sensor format collectively influence perspective, framing and depth of field.

Krea 2 does not operate as a conventional optical simulation. Instead, it interprets photographic terminology through learned visual associations.

Consequently, instructions such as:

`Shot with an 85mm lens at f/2.8`

may encourage the characteristics of portrait photography and relatively shallow depth of field, but they do not guarantee an optically accurate reproduction.

The distinction is between two complementary types of instruction:

|Instruction|Purpose|
|---|---|
|Photographic parameters|Establish the intended photographic characteristics|
|Visual descriptions|Specify how those characteristics should appear in the generated image|

Using both reduces ambiguity.

## 2. Why Shallow Depth of Field Is Not Sufficiently Precise

The instruction:

`Shallow depth of field`

communicates a general photographic effect but leaves several important decisions to the model.

These include:

- Which elements should remain sharply focused.
    
- Whether the entire subject or only the face should be sharp.
    
- Where background defocusing should begin.
    
- How strongly the background should be blurred.
    
- Whether background objects should remain recognisable.
    
- Whether blur should increase with distance.
    

Krea 2 may produce a visually plausible shallow-depth-of-field image without matching the intended distribution of sharpness.

### Increasing levels of specificity

**Level 1 — General photographic terminology**

`Shallow depth of field.`

Establishes the broad photographic effect.

**Level 2 — Subject separation**

`Shallow depth of field, subject sharply focused against a softly blurred background.`

Identifies the principal focus relationship.

**Level 3 — Explicit focus hierarchy**

`Subject in crisp focus from head to toe, with the bed immediately behind them gently softened but still recognisable, and furnishings farther into the room progressively more defocused.`

Defines the intended sharpness of individual scene elements and the relationship between them.

The third approach provides the greatest descriptive control, although the model may not follow every instruction precisely.

## 3. Practical Example — Subject Standing in Front of a Bed

### Objective

Generate a realistic photograph featuring a subject standing in front of a bed.

The desired result is:

- The subject remains crisp and sharply focused.
    
- The bed is softly defocused but clearly recognisable.
    
- Background furnishings become progressively softer.
    
- The background retains sufficient detail to establish the bedroom environment.
    
- The overall effect resembles a conventional portrait photographed with a 50mm or 85mm lens at approximately f/2.8–f/3.5.
    

### Recommended photographic description

`Photographed with an 85mm lens at f/2.8, precise focus on the subject, gently softened background with the bed remaining recognisable.`

### More controlled description

`Photographed on a full-frame camera with an 85mm portrait lens at f/2.8. The subject remains crisp and sharply focused, with clear facial features, clothing details and natural textures. The bed immediately behind the subject is gently defocused but remains recognisable, while furnishings farther into the bedroom become progressively softer. Natural, restrained background blur preserves the environmental context without distracting from the subject.`

The second version is more explicit about the desired visual effect and avoids relying entirely on the model's interpretation of the aperture setting.

### Complete example prompt

The following is an illustrative prompt for testing, not a verified optimal Krea 2 configuration.

```
A realistic photograph of an adult woman standing naturally in front of a neatly made bed in a contemporary bedroom. She wears casual everyday clothing and faces the camera with a relaxed expression. The bed is visible directly behind her, with bedside tables, lamps and additional bedroom furnishings extending farther into the room.

Photographed from eye level using a full-frame camera with an 85mm portrait lens at f/2.8. Natural perspective and balanced composition, with the subject fully visible in the frame.

The subject remains crisp and sharply focused from head to toe, preserving clear facial features, natural skin texture and fine clothing details. The bed behind her is gently softened but remains clearly recognisable, while furnishings farther into the room become progressively more defocused. Subtle, natural background blur creates subject separation without obscuring the bedroom environment.

Soft natural daylight illuminates the room, with realistic shadows, restrained contrast, natural colour reproduction and physically plausible lighting. Photorealistic appearance with natural textures and proportions.
```

## 4. Lens and Aperture Selection

Conventional photographic knowledge remains useful when choosing the initial parameters.

For the bedroom example:

|Configuration|Intended photographic character|
|---|---|
|50mm, f/2.8|Natural-looking environmental portrait with moderate background separation|
|50mm, f/3.5|Slightly greater apparent depth of field under otherwise equivalent conditions|
|85mm, f/2.8|Portrait-oriented framing with pronounced potential for subject separation|
|85mm, f/3.5|Portrait-oriented framing with somewhat more restrained defocusing|

These are photographic expectations, not verified Krea 2 behaviours.

Actual depth of field also depends on subject distance, focus distance and sensor format. Perspective is determined by camera position rather than focal length alone.

When prompting Krea 2, numerical camera settings should therefore be treated as guidance rather than guaranteed optical constraints.

### Avoiding excessive blur

For scenes where environmental context matters, avoid unnecessarily strong descriptions such as:

- `Extreme bokeh`
    
- `Completely blurred background`
    
- `Ultra-shallow depth of field`
    

Such instructions may encourage the model to obscure important background details.

Prefer descriptions such as:

`Gently softened background, recognisable furniture, restrained depth-of-field effect.`

## 5. Prompt Structure

A useful Krea 2 prompt structure is:

1. **Subject and environment:** Describe what appears in the scene.
    
2. **Composition and camera position:** Define viewpoint, framing and subject placement.
    
3. **Photographic parameters:** Specify focal length, aperture and other relevant photographic characteristics.
    
4. **Focus hierarchy:** Explicitly identify sharp and defocused elements.
    
5. **Lighting and rendering:** Describe illumination, textures, colour and overall realism.
    

Photographic instructions can be grouped towards the end of a prompt, after the scene description.

This is an organisational convention, not a verified requirement that Krea 2 assigns greater importance to instructions appearing in a particular position.

## 6. Controlled Testing in ComfyUI

The existing minimal Krea 2 workflow provides a useful environment for investigating how the model interprets photographic terminology.

A controlled experiment can compare three prompting approaches.

### Test A — Photographic parameters only

Use the same base scene description with:

`Photographed with an 85mm lens at f/2.8.`

### Test B — Visual description only

Use the same base scene description with:

`The subject remains sharply focused while the bed behind them is softly defocused but recognisable.`

### Test C — Combined approach

Use the same base scene description with:

`Photographed with an 85mm lens at f/2.8. The subject remains sharply focused while the bed behind them is softly defocused but recognisable.`

### Test conditions

Keep the following unchanged between generations:

- Base scene description.
    
- Model checkpoint and LoRAs.
    
- Resolution.
    
- Seed.
    
- Sampler and scheduler.
    
- Sampling steps.
    
- CFG and other generation parameters.
    
- Prompt enhancement settings, preferably bypassed for direct comparison.
    

The existing base Krea 2 workflow has previously been used successfully with CFG 1.0, eight steps and a fixed seed. Those values are historical test conditions, not universal recommendations.

### Evaluation criteria

Compare the generated images for:

- Subject sharpness.
    
- Background blur intensity.
    
- Bed recognisability.
    
- Progressive defocusing with distance.
    
- Consistency of subject placement and composition.
    
- Whether the photographic parameters contribute a visible effect beyond the explicit focus description.
    

Using a fixed seed improves comparison by holding the initial noise constant, but does not guarantee identical composition when the prompt changes.

Multiple seeds would help determine whether an observed effect is consistent rather than specific to one generation.

### Status

**Proposed experiment — not yet executed or validated.**

No controlled results have been established demonstrating how accurately Krea 2 responds to specific focal lengths or f-numbers.

## 7. Limitations and Caveats

### Photographic terminology is not an optical guarantee

Aperture and focal-length specifications may influence the generated aesthetic without producing physically accurate depth-of-field relationships.

### Focus descriptions may compete with scene geometry

A subject standing very close to a bed may naturally occupy nearly the same focus region.

Requesting a sharply focused subject and noticeably defocused bed could therefore create a result that is not strictly consistent with the implied photographic setup.

### Full-body sharpness can be challenging

A request for a fully sharp subject combined with strongly blurred surroundings may not always correspond to the stated lens, aperture and distances.

Explicit visual instructions can help establish the desired outcome, but do not guarantee physical consistency.

### Additional photographic terminology is not always beneficial

Camera brands, sensor models and elaborate lens descriptions should only be included when they serve a specific visual purpose.

Unnecessary detail can introduce competing aesthetic associations.

## 8. Conclusions

The key distinction is between **describing a photographic technique** and **describing its visible consequences**.

`Shallow depth of field` establishes a general effect.

`85mm lens at f/2.8` provides a more specific photographic reference.

`Subject sharply focused, bed gently defocused but recognisable, distant furnishings progressively softer` communicates the desired visual outcome.

Combining these approaches gives Krea 2 clearer guidance while retaining the vocabulary of conventional photography.

**Practical principle: Photographic terminology establishes the aesthetic; describing the visible result establishes the intent.**

The effectiveness of individual focal-length and aperture specifications remains an open question for controlled testing in the existing ComfyUI workflow.