---

type: reference  
status: active  
created: 2026-10-18  
source: ChatGPT — ComfyUI / Image Generation project  
tags:

- comfyui
    
- wan
    
- wan-2-2
    
- video-generation
    
- image-to-video
    
- diffusion
    
- ai-filmmaking
    

purpose: knowledge
---
> [!abstract] Overview  
> WAN 2.2 is a generative video model family capable of producing moving images from text and/or image conditioning. This article introduces the principles behind video generation, focusing on the WAN 2.2 A14B image-to-video architecture, its two noise experts, sampling, acceleration, performance and practical limitations.
> 
> The examples and performance measurements come from experiments using ComfyUI Desktop and an NVIDIA RTX 5070 Ti with 16 GB VRAM.

## 1. Understanding Video Diffusion

Traditional video production records a sequence of images representing movement through time.

Diffusion-based video generation works differently. Rather than photographing movement, the model progressively transforms noise into a representation of a video sequence, guided by conditioning such as text, images and other supported inputs.

The model must produce not only convincing individual frames but also plausible relationships between frames.

This introduces additional challenges compared with still-image generation:

- **Spatial consistency:** Objects must retain plausible shapes and positions.
    
- **Temporal consistency:** Details should remain stable between frames.
    
- **Motion coherence:** Movement should follow believable trajectories.
    
- **Identity preservation:** Characters should remain recognisable.
    
- **Camera consistency:** Camera movement should not accidentally alter scene geometry.
    
- **Continuity:** Clothing, props, lighting and environmental details should remain consistent.
    

A visually convincing individual frame does not guarantee a convincing video.

### 1.1 Latent video generation

WAN operates on compressed latent representations rather than directly constructing every output pixel during diffusion.

Conceptually:

1. Construct the initial video latent and conditioning.
    
2. Apply diffusion sampling across a sequence of noise levels.
    
3. Progressively develop spatial and temporal structure.
    
4. Decode the completed latent into video frames using a VAE.
    
5. Encode the resulting frames into a playable video.
    

The latent representation is a compressed mathematical description of the video, not a collection of finished images.

This distinction becomes particularly important when examining WAN 2.2's dual-expert architecture.

---

## 2. Text-to-Video and Image-to-Video

WAN supports different generation tasks, depending on the model variant.

### 2.1 Text-to-video (T2V)

Text-to-video generation begins without a supplied source image.

The prompt describes the intended scene, including its subjects, environment, actions and camera behaviour.

The model must invent the visual composition as well as the motion.

**Advantages**

- Greater freedom to explore new scenes.
    
- No source image required.
    
- Useful for conceptual experimentation.
    

**Limitations**

- Less direct control over initial composition.
    
- Character identity can be difficult to preserve.
    
- Environments and wardrobe may vary between generations.
    
- Establishing continuity across separate shots is challenging.
    

### 2.2 Image-to-video (I2V)

Image-to-video generation uses a supplied image to establish the initial visual state.

The image provides important information about:

- Character appearance and wardrobe.
    
- Composition and framing.
    
- Environment and lighting.
    
- Object positions.
    
- Initial pose and visual style.
    

The prompt primarily guides what should happen after that initial state.

**Advantages**

- Greater control over the starting composition.
    
- Better opportunity to preserve established characters and locations.
    
- Works naturally with separately generated keyframes.
    
- Particularly suitable for shot-based filmmaking.
    

**Limitations**

- The source image does not completely constrain subsequent frames.
    
- The model can invent unseen body parts or environmental details.
    
- Character identity can drift during substantial movement.
    
- Prompts cannot guarantee a fixed pose or camera.
    

### Practical production principle

For continuity-sensitive filmmaking, prefer:

**Approved still image → WAN I2V → generated shot → nonlinear editor**

This separates visual design from motion generation.

The image establishes _what the scene looks like_.

WAN determines _how the scene develops over time_.

---

## 3. WAN 2.2 A14B — Dual-Expert Architecture

WAN 2.2 A14B uses a mixture-of-experts approach involving two diffusion experts specialised for different noise regimes.

These are commonly referred to as:

- **High-noise expert**
    
- **Low-noise expert**
    

The two experts operate at different stages of the diffusion process.

They are not two separate video generators producing independent clips.

### 3.1 High-noise expert

The high-noise expert handles the earlier portion of sampling, when the latent contains relatively little resolved visual information.

At this stage, the model develops much of the broad structure of the generated sequence.

Conceptually, this includes:

- Overall scene composition.
    
- Large-scale subject movement.
    
- Broad camera behaviour.
    
- Spatial relationships.
    
- Temporal organisation.
    
- General visual structure.
    

This does not mean every motion decision is permanently fixed during the high-noise stage.

Subsequent denoising can still affect the result.

### 3.2 Low-noise expert

The low-noise expert operates later in the diffusion process, when the latent is closer to the final representation.

Its role is associated with refining the emerging sequence.

Conceptually, this includes:

- Fine visual details.
    
- Surface textures.
    
- Edges and facial structure.
    
- Appearance refinement.
    
- Temporal consistency of details.
    

Both experts contribute to the completed result.

### 3.3 Expert handover

The high-noise expert produces a partially denoised latent.

That latent is passed to the low-noise expert, which continues the same diffusion trajectory.

```
Initial noisy video latent
           |
           v
    HIGH-NOISE EXPERT
    Early denoising
           |
           v
   Partially denoised latent
           |
           v
     LOW-NOISE EXPERT
     Later denoising
           |
           v
    Completed video latent
           |
           v
        VAE Decode
           |
           v
       Video frames
```

The handover is a **latent state**, not an intermediate video.

### 3.4 Experimental finding: decoding the handover

We tested whether the high-noise expert's output could be decoded independently to provide an inexpensive preview.

Two experiments were performed:

|Configuration|High-noise sampling|Observed result|
|---|---|---|
|Turbo|2 of 4 steps|Predominantly noise|
|Native|10 of 20 steps|Noise with very faint structural information|

The native experiment required approximately five minutes.

**Conclusion:** Directly decoding the intermediate high-noise latent did not produce a useful rough-cut preview in our tested workflow.

This does not mean the high-noise expert failed. The intermediate representation simply had not completed the diffusion process.

The experiment also demonstrated why the low-noise expert cannot be treated as an optional visual-polishing filter.

---

## 4. Sampling Fundamentals

Sampling is the iterative process that transforms the noisy latent into the final video representation.

Several parameters influence the result.

### 4.1 Sampling steps

Steps determine how many denoising iterations are performed.

In our initial workflow, the native configuration used:

- Total steps: `20`
    
- High-noise phase: steps `0–10`
    
- Low-noise phase: steps `10–20`
    

These values describe our tested configuration, not universal requirements for every WAN 2.2 workflow.

More steps do not automatically guarantee a better result. The appropriate number depends on the model, sampler, scheduler and any acceleration method.

### 4.2 CFG — Classifier-Free Guidance

CFG controls the influence of conditioning during sampling.

Broadly:

- Lower CFG permits greater freedom relative to the conditioning.
    
- Higher CFG increases guidance pressure.
    
- Excessively strong guidance can introduce unwanted visual behaviour.
    

Our tested configurations used:

|Mode|CFG|
|---|---|
|Native|3.5|
|LightX2V Turbo|1.0|

These values should not be interchanged casually.

Acceleration LoRAs may expect substantially different sampling conditions from the unaccelerated model.

### 4.3 Sampler and scheduler

The sampler defines the numerical method used to advance denoising.

The scheduler determines the progression through noise levels.

Together, they influence how the diffusion trajectory develops.

Different sampler/scheduler combinations may produce different motion, composition and detail even with otherwise identical settings.

When comparing generation settings, preserve the sampler and scheduler unless they are the variables being tested.

### 4.4 Random seed

The seed initialises the random state used by generation.

Preserving the seed helps make controlled comparisons possible.

However, a matching seed does not guarantee matching video when other variables change.

Changing the model, LoRA, sampling steps, dimensions or conditioning can alter the entire generation trajectory.

**Experimental rule:** Change one variable at a time whenever possible.

---

## 5. LightX2V and Four-Step Acceleration

LightX2V acceleration uses LoRAs designed to allow generation with substantially fewer sampling steps.

In our WAN 2.2 workflow, separate acceleration LoRAs are applied to the high-noise and low-noise experts.

The tested Turbo configuration used:

- Total steps: `4`
    
- High-noise phase: `2` steps
    
- Low-noise phase: `2` steps
    
- CFG: `1.0`
    

This substantially reduces generation time.

### 5.1 Native versus Turbo

Native generation uses a longer diffusion process.

Turbo generation uses an accelerated process designed for fewer steps.

The two approaches can produce different results even when supplied with the same source image and prompt.

### 5.2 Experimental finding: Turbo is not a faithful native preview

We generated the same character scene using both approaches.

Turbo produced coherent motion and was substantially faster.

However, native generation introduced camera movement and facial-expression changes that were not accurately predicted by the Turbo result.

The native sequence developed a pronounced camera push and a more aggressive facial expression.

**Conclusion:** Turbo is useful for exploring prompts, approximate motion possibilities and workflow behaviour, but should not be considered a guaranteed low-quality preview of the eventual native generation.

A useful mental model is:

**Turbo and native are different takes of the same scene, not necessarily different-quality renders of an identical take.**

This distinction matters when using Turbo for production planning.

---

## 6. Video Resolution and Aspect Ratio

Video generation is computationally expensive because it must process spatial information across multiple frames.

Resolution therefore has a substantial impact on performance.

### 6.1 Pixel count

The spatial pixel count is:

`width × height`

Our experiments included:

|Resolution|Pixels per frame|Relative to 640×640|
|---|---|---|
|640×640|409,600|100%|
|448×640|286,720|70%|
|320×448|143,360|35%|
|224×320|71,680|17.5%|

These ratios describe pixel counts, not guaranteed runtime or VRAM scaling.

Actual computational cost also depends on latent dimensions, temporal processing, model loading, attention implementation and other runtime factors.

### 6.2 Preserve the source image's aspect ratio

Our original Nate keyframe was approximately 832×1216, a portrait composition.

The first WAN experiments used 640×640.

This changed the intended framing and allocated substantial computation to an unsuitable square composition.

Moving to 448×640 preserved a much more appropriate portrait framing while reducing the spatial workload.

**Lesson:** Choose a generation resolution based on the source composition and intended shot, not simply the largest dimensions the GPU can handle.

### 6.3 Lower-resolution experiments

We also tested very small Turbo generations.

At 224×320, facial information became too degraded for reliable evaluation.

At 320×448, the face remained more readable and generation was very fast.

This made 320×448 useful for inexpensive exploration, although it did not eliminate the differences between Turbo and native motion.

---

## 7. Frame Rate and Duration

Video duration depends on the number of frames and the playback frame rate.

A basic relationship is:

`duration ≈ frame count / frames per second`

Our initial workflow used 16 FPS.

The template calculated frame count using:

`length = duration × FPS + 1`

For a nominal five-second generation:

`5 × 16 + 1 = 81 frames`

The additional frame is a convention used by this workflow; it should not be assumed to apply to every WAN implementation.

At 16 FPS, 81 frames represent 80 frame-to-frame intervals across five seconds.

The resulting media file's reported playback duration can depend on how frames are encoded and timestamped.

### 7.1 Why duration matters

Longer sequences require the model to maintain coherent motion and appearance over more frames.

Potential problems include:

- Gradual identity drift.
    
- Unexpected pose changes.
    
- Camera movement not requested in the prompt.
    
- Clothing or prop changes.
    
- Invented scene details.
    
- Increasing divergence from the starting image.
    

Shorter clips can reduce the amount of unsupported motion the model must invent.

For shot-based production, several short clips may be easier to control than one long continuous generation.

---

## 8. Performance Measurements

The following results were observed on our local workstation.

### Test environment

- GPU: NVIDIA RTX 5070 Ti
    
- VRAM: 16 GB
    
- System RAM: 32 GB
    
- Application: ComfyUI Desktop
    
- Model family: WAN 2.2 A14B I2V
    
- Frame rate: 16 FPS
    
- Nominal duration: 5 seconds
    

### Observed generation times

|Mode|Resolution|Approximate runtime|
|---|---|---|
|Native|640×640|10m 40s|
|Native|448×640|5m 51s|
|Turbo|640×640|96s|
|Turbo|320×448|20s|

These are measurements from individual experiments, not formal hardware benchmarks.

They should not be interpreted as guaranteed performance for other prompts, model versions or system configurations.

### Key observations

Reducing native resolution from 640×640 to 448×640 produced a substantial runtime improvement while better preserving the original portrait composition.

Turbo made rapid experimentation practical, particularly at smaller resolutions.

However, generation speed alone did not determine which mode was more useful for a particular filmmaking task.

**The fastest generation is not necessarily the most useful generation.**

---

## 9. Prompting for Motion

Image-generation prompts and video-generation prompts serve different purposes.

A still-image prompt primarily describes visual appearance.

A video prompt must also describe changes over time.

Useful categories include:

- **Subject:** Who or what is present.
    
- **Initial state:** How the subject begins.
    
- **Action:** What changes during the clip.
    
- **Camera:** Whether the viewpoint moves.
    
- **Environment:** What remains visible and consistent.
    
- **Atmosphere:** Lighting, energy, weather and other effects.
    
- **Temporal progression:** How the action develops.
    

### 9.1 Describe achievable actions

WAN can produce convincing short actions, but precise behavioural constraints are not guaranteed.

Our Nate experiment began with a crouched character preparing magic.

Despite explicit instructions to remain crouched, keep his head lowered and avoid looking at the camera, native generation repeatedly introduced unwanted movement after approximately three seconds.

The model raised the character, changed his gaze and modified his expression.

Both detailed and simplified prompts failed to eliminate the behaviour reliably.

### 9.2 Why text alone can fail

A video model learns statistical patterns of motion and visual transitions.

It does not operate like a traditional animation rig with explicit skeletal constraints.

A text instruction such as “remain crouched” influences the generation, but does not mechanically lock the character's pose.

The same applies to:

- Fixed camera positions.
    
- Unchanging facial expressions.
    
- Stable clothing.
    
- Precisely timed gestures.
    
- Character interactions.
    

When exact motion is required, additional conditioning or a different shot design may be necessary.

### 9.3 Design around model strengths

Our first Nate generation contained approximately three seconds of useful footage before significant drift occurred.

Rather than discard the entire clip or demand five flawless seconds, we recognised that the usable portion could become a complete shot.

The next shots could show other characters reacting before returning to Nate in a new visual state.

This allows editing to bridge transitions that the model struggles to generate directly.

> [!tip] Production principle  
> Generate the action the shot needs, not every action that happens between shots.

---

## 10. Image-to-Video and Character Continuity

A source image is an important visual anchor, but not a complete identity-locking mechanism.

As the generated character moves, the model may need to invent visual information that was not present in the source image.

For example, a crouched character standing upright exposes new information about:

- Torso proportions.
    
- Coat length and folds.
    
- Legs and footwear.
    
- Body posture.
    
- Facial angle.
    
- Hidden surfaces.
    

The model must estimate these details.

This creates opportunities for continuity errors.

### 10.1 Establish the initial state deliberately

For our planned final Nate shot, the intended action is magical escalation while Nate stands upright.

Instead of asking WAN to transform a crouched Nate into a standing Nate, we decided to create a new approved still with Nate **already standing**.

WAN then only needs to animate the magical effects and expression changes.

This reduces the number of simultaneous transformations required.

### 10.2 Separate identity from performance

A practical pipeline is:

1. Generate or select a canonical character image.
    
2. Establish the desired pose, wardrobe and environment.
    
3. Correct visual defects before animation.
    
4. Supply the approved keyframe to WAN.
    
5. Animate the specific performance required.
    
6. Edit the usable portion into the scene.
    

Character LoRAs can help maintain identity during still-image creation, but they do not automatically solve temporal consistency in a separate video model.

---

## 11. Native, Turbo and Production Strategy

Different generation modes are useful at different stages.

|Task|Practical approach|
|---|---|
|Explore whether a prompt produces plausible movement|Turbo|
|Check approximate scene behaviour quickly|Turbo at reduced resolution|
|Evaluate final facial expression and camera behaviour|Native|
|Produce an intended final shot|Native, subject to visual assessment|
|Verify exact continuity|Inspect actual generated frames|
|Build a longer scene|Combine short shots in an NLE|

Turbo should be treated as an experimentation tool, not a promise of what native will produce.

Native generation is not automatically correct either. It must still be judged on the resulting footage.

---

## 12. Common Failure Modes

|Failure|Likely contributing factor|Practical response|
|---|---|---|
|Character changes pose unexpectedly|Learned motion patterns, weak pose constraints|Shorten shot or provide a new starting keyframe|
|Camera moves unexpectedly|Generated camera-motion interpretation|Evaluate native output; consider stronger conditioning|
|Face changes expression|Temporal generation introduces performance changes|Use shorter shots or revised starting image|
|Character identity drifts|New viewpoints require invented visual information|Improve source frame and reduce unsupported transformations|
|Turbo differs from native|Different sampling/acceleration behaviour|Treat as separate takes|
|Intermediate latent decodes as noise|Denoising is incomplete|Finish sampling before evaluating video|
|Generation is too slow|Spatial/temporal workload|Reduce dimensions or duration|
|Source framing is damaged|Inappropriate aspect ratio|Match source composition more closely|

These are practical diagnostic starting points, not guaranteed explanations for every failure.

---

## 13. Recommended Learning Workflow

For learning WAN 2.2 efficiently:

1. Begin with a visually stable source image.
    
2. Choose dimensions appropriate to its aspect ratio.
    
3. Start with a short duration and a simple action.
    
4. Use Turbo for inexpensive exploration.
    
5. Freeze the seed when testing individual parameter changes.
    
6. Evaluate native generation independently.
    
7. Inspect temporal consistency, not merely the first and last frames.
    
8. Keep successful portions even if the complete clip is imperfect.
    
9. Record actual runtime and settings.
    
10. Move to more complex motion only after understanding simpler behaviour.
    

The objective is to build an understanding of how the model responds to changes in conditioning, sampling and scene design.

---

## 14. Key Lessons from Our Experiments

> [!important] Findings  
> **1. WAN is a generative system, not a deterministic animation rig.** Text influences motion but does not guarantee it.
> 
> **2. High-noise and low-noise experts are complementary stages.** Their handover is an unfinished latent, not a preview video.
> 
> **3. Turbo and native can behave differently.** Turbo is useful for exploration but cannot be trusted to predict final camera movement or performance.
> 
> **4. Resolution is a creative decision as well as a performance decision.** Correct aspect ratio can improve framing while reducing computation.
> 
> **5. The starting image matters enormously.** It establishes the visual state the model must develop.
> 
> **6. Short, purposeful shots are easier to manage.** Editing can bridge actions that are difficult to generate reliably.
> 
> **7. Continuity should be designed before generation.** Do not rely on the video model to invent consistent character and environment details.

The broader principle guiding our filmmaking experiments is:

**AI should improvise the performance. It shouldn't improvise the continuity.**

---

## 15. Related Notes

- [[WAN 2.2 — ComfyUI Workflow Architecture]] — Dual-expert implementation, KSamplers, switches, sampling stages and Turbo configuration.
    
- [[ComfyUI — Inpainting and Scene Geometry]] — Masks, reconstruction, conditioning and perspective.
    
- [[ComfyUI — Environment Plates and LoRA Strategy]] — Canonical environments, character assets and compositing.
    
- [[AI Filmmaking — Shot Construction and Continuity]] — Shot planning, keyframes, editing and continuity management.
    

---

## 16. References

- [WAN 2.2 — Official GitHub repository](https://github.com/Wan-Video/Wan2.2)
    
- [ComfyUI — Official documentation](https://docs.comfy.org/)
    
- [LightX2V — GitHub repository](https://github.com/ModelTC/LightX2V)
    

Performance figures and production observations in this article originate from our local ComfyUI experiments rather than published benchmarks.