---
type: reference
status: active
created: 2026-10-17
source:
tags:
  - krea-2
  - lighting
  - cinematic
  - image-generation
  - photography
  - comfyui
---
## Overview

Getting dark cinematic lighting in Krea 2 is primarily about controlling the relationship between light, shadow, contrast and exposure, rather than simply asking for a dark image.

And this connects rather nicely to our recent conversation about lenses and depth of field: describe the photographic effect you want, not just the technical terminology.

The trick is to make Krea understand that the scene is deliberately lit, rather than simply underexposed.

## 1. Think like a cinematographer

There are four things I'd concentrate on:

- Motivated lighting: Give the light a believable source, such as a window, candle, streetlamp or moonlight.
    
- Low-key lighting: Most of the scene is in shadow, with selected areas illuminated.
    
- Directional lighting: Light comes predominantly from one direction, creating shape and depth.
    
- Controlled exposure: The subject is correctly exposed within an otherwise dark environment.
    

That last distinction is particularly important. You don't necessarily want a dark subject. You want a subject emerging from darkness.

![Women face in low key](https://images.openai.com/static-rsc-4/y02tK7AlzoVuqufdY0Vi5VDwZt2R8CqlOKaZb5msAUlriyqjRzykX7jRbSQiBPOeHhAEWte5qPbQZjC7nlVOsz87TfJli17fRW8lArWnRmvnI2sTcsZm6BbEeQtqIJrgD-XsaBJ3u8CIubaOpJ6odzrZOCAKkOhzzFCrjPKAsdNz9SV2buNduhxVh8PNFlzV?purpose=inline)

Directional key light with deep shadows

![AI Art by Henrik Harbin | Deep Dream Generator](https://images.openai.com/static-rsc-4/3JcDoBAMEoldtXEkv6uUL5OGfnkxREiBu9fTmyStGo4u13e_VqM5AIzZwuELowbQAZi0KFQMUAMU46-cG_AUHKjpGqsFf0cYBqSXi7QZFGuQ3OZAvcJQVdZKSyvmU8dZttwzZX9Q_Jll-ldUhCHvBjKfQGoctoMiFUr36zGUPgo?purpose=inline)

Motivated environmental lighting

## 2. The lighting vocabulary that matters

|Technique|What it achieves|
|---|---|
|Low-key lighting|Dark overall scene with selective illumination|
|Chiaroscuro|Strong separation between illuminated and shadowed areas|
|Negative fill|Deepens shadows by reducing bounced light|
|Light falloff|Creates a gradual transition into darkness|
|Rim lighting|Separates the subject from a dark background|
|Practical lighting|Visible sources such as lamps or candles|
|Crushed blacks|Creates near-black shadows, sacrificing some detail|
|Subtle atmospheric haze|Reveals light beams and creates depth|

For realistic cinematic results, I'd initially favour deep but detailed shadows over crushed blacks. You can always darken the image further, but recovering detail that Krea never generated is another matter.

## 3. A complete Krea 2 example

Let's use our recent bedroom scenario: an adult subject standing in front of a bed, with the subject sharp and the background softly defocused.

A cinematic photographic still of an adult woman standing in front of a bed in a dark, elegant bedroom at night. She is positioned slightly off-centre, facing the camera with a calm, serious expression. The room is spacious, with dark wooden furniture, muted fabrics and a large window to her left.

The scene uses carefully controlled low-key cinematic lighting. A narrow shaft of cool moonlight enters through the window from camera left, softly illuminating one side of her face, shoulders and upper body. The opposite side falls into deep, natural shadow, with very little ambient fill light. A faint warm bedside lamp provides subtle background illumination without brightening the entire room.

The subject is correctly exposed against the much darker surroundings, with natural skin tones, restrained highlights and rich shadows retaining subtle texture. Light falls off rapidly into the background, leaving the far corners of the room almost completely dark. The lighting creates strong dimensional separation between the subject and her surroundings, with subtle chiaroscuro and a restrained cinematic contrast ratio.

Photographed with the perspective of an 85mm portrait lens at f/2.8, the subject remains crisp and sharply focused while the bed and distant furnishings dissolve into a soft, natural background blur. Muted blue-grey shadows, subtle warm amber accents, realistic filmic tonal response, delicate film grain, understated colour grading, atmospheric cinematic realism.

Notice that we haven't simply written dark, moody, cinematic repeatedly. We've described exactly where the light originates, what it illuminates, what remains in shadow, and how the exposure should behave.

That approach is consistent with Krea's published guidance, which favours natural-language descriptions and specific visual direction.

![](https://www.google.com/s2/favicons?domain=https://github.com&sz=32)

GitHub

+1

## 4. How I'd test this in your existing workflow

Given our established Krea 2 base workflow, I'd avoid changing the sampler, CFG, steps or adding LoRAs initially.

Keep your existing seed fixed and make three generations using the same complete prompt, changing only the lighting instructions between tests.

|Test|Lighting treatment|What to examine|
|---|---|---|
|A|Soft directional moonlight|Shadow depth and subject exposure|
|B|Hard directional moonlight|Stronger contrast and defined shadows|
|C|Soft moonlight with minimal ambient fill|Whether the background becomes convincingly darker|

One potential complication is Krea interpreting cinematic as an instruction to make everything attractively illuminated. If that happens, I would explicitly reinforce the absence of ambient illumination rather than immediately trying to manipulate CFG.

And because your workflow already has an image-processing path, we can consider a final colour-grading stage later if the generated lighting is structurally correct but the overall exposure remains too bright.

One question, though: when you say dark cinematic, are you thinking more along the lines of film noir with dramatic pools of light and near-black shadows, or modern cinematic darkness where everything is subdued but you can still see detail throughout the scene?

They're quite different looks, and the distinction will substantially influence how we prompt Krea.