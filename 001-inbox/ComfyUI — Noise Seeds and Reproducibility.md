---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- diffusion-models
    
- noise-seeds
    
- latent-space
    
- reproducibility
    

---
## Overview

In diffusion-based image generation, a **seed** is an integer used to initialise a pseudorandom-number generator (PRNG), which produces the noise used as the starting point for image generation.

The seed does not contain the noise itself. Instead, it provides a reproducible way to generate that noise.

The fundamental distinction is:

> **The seed determines the starting conditions. The model, conditioning, LoRAs, sampler, scheduler and other generation parameters determine how those conditions develop into the final image.**

Consequently, using the same seed does not guarantee the same final image when other generation parameters change.

## 1. How a Seed Works

A typical text-to-image diffusion process follows this sequence:

```
Seed
  |
  v
Pseudorandom Number Generator
  |
  v
Initial Noise Tensor
  |
  v
Diffusion Sampling
  |
  |-- Model / Checkpoint
  |-- LoRAs
  |-- Prompt Conditioning
  |-- Sampler
  |-- Scheduler
  |-- CFG
  |-- Sampling Steps
  |
  v
Final Latent Representation
  |
  v
VAE Decoder
  |
  v
Generated Image
```

The seed is supplied to a pseudorandom-number generator, which produces a tensor containing numerical noise values.

During diffusion sampling, the model progressively transforms the noisy latent representation into a structured representation of an image.

The VAE decoder then converts the resulting latent into image pixels.

### Seed versus noise

These concepts are related but distinct:

|Concept|Description|
|---|---|
|Seed|Integer used to initialise pseudorandom generation|
|PRNG|Algorithm producing reproducible pseudorandom values|
|Noise tensor|Multidimensional array of generated noise values|
|Latent|Numerical representation processed by the diffusion model|
|Sampler|Algorithm controlling the iterative denoising process|
|Scheduler|Determines the noise levels or timesteps used during sampling|

**A seed is not an image, a latent representation or a stored noise pattern.** It is an input used to generate the initial noise.

## 2. Reproducibility

When the relevant generation conditions are identical, reusing the same seed should reproduce the same initial noise tensor.

For example:

```
Seed:        123456
Model:       Model A
Resolution:  1024 x 1024
Sampler:     Euler
Scheduler:   Normal
Steps:       28
CFG:         4.0
Prompt:      Unchanged
```

Repeating this generation under equivalent conditions should produce the same result, subject to implementation-specific nondeterminism.

However, the seed alone does not guarantee reproducibility.

Other relevant conditions include:

- The pseudorandom-number generator and its implementation.
    
- The shape and generation method of the noise tensor.
    
- The checkpoint and any active LoRAs.
    
- Prompt conditioning and guidance settings.
    
- Sampler, scheduler and sampling steps.
    
- Software versions, numerical precision and execution environment.
    

Different implementations may generate different noise tensors from the same integer seed.

### Important distinction

**Same seed does not universally mean identical noise.**

It means identical noise only when the relevant noise-generation conditions are also equivalent.

## 3. Using the Same Seed Across Different Models and Settings

An initial noise tensor can be processed using different diffusion configurations, provided their latent representations and sampling requirements are compatible.

For example:

```
                     Seed 123456
                          |
                          v
                  Initial Noise
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
           Model A     Model B     Model A
           Euler       Euler       DPM++
           No LoRA     No LoRA     + LoRA
              |           |           |
              v           v           v
           Image A     Image B     Image C
```

Assuming the same initial noise tensor is used, all three generations begin from equivalent noise but follow different denoising processes.

The resulting images may differ significantly.

### Checkpoints

Different checkpoints contain different learned parameters and may interpret identical noise and conditioning differently.

Changing the checkpoint can affect:

- Composition and subject placement.
    
- Facial characteristics and anatomy.
    
- Lighting and colour.
    
- Artistic style.
    
- Background interpretation.
    
- Fine details.
    

A shared seed does not guarantee visual similarity between checkpoints.

### LoRAs

LoRAs modify the behaviour of the underlying model.

Enabling a LoRA while preserving the initial noise allows its influence to be investigated without simultaneously introducing a different random starting point.

For example:

```
Generation A:
  Checkpoint: Model A
  LoRA:       Disabled
  Seed:       123456

Generation B:
  Checkpoint: Model A
  LoRA:       Enabled
  Seed:       123456
```

Assuming everything else remains unchanged, the resulting differences can be attributed to enabling the LoRA.

This is particularly useful when evaluating style, character, anatomy or detail-enhancement LoRAs.

### Samplers and schedulers

Different samplers and schedulers can produce different results from identical initial noise.

They determine how the latent evolves during sampling, including which noise levels are evaluated and how model predictions are applied.

Changing these parameters may alter composition, detail and visual characteristics.

### CFG and sampling steps

Changing CFG modifies how conditioning influences the denoising process.

Changing the number of steps modifies the sampling trajectory or its numerical resolution.

Both can change the final result without changing the initial noise.

## 4. Seed Numbers Are Not Visually Sequential

A common misconception is that numerically adjacent seeds should produce visually similar images.

For example:

```
Seed 123456
Seed 123457
```

These values are adjacent integers, but their generated noise tensors are not necessarily similar.

Conceptually:

```
Seed 123456 --> Noise Field A
Seed 123457 --> Noise Field B
```

A well-behaved PRNG does not make consecutive seeds correspond to progressively changing images.

Therefore:

> **Incrementing a seed is not equivalent to making a small variation of an existing image.**

To produce controlled variations, other techniques may be more appropriate, such as image-to-image generation with reduced denoising strength or controlled modifications to the noise tensor.

## 5. Seeds and Image Composition

Although noise appears random, its influence extends beyond fine texture.

Changing the initial noise can affect high-level image characteristics, including:

- Subject position and pose.
    
- Camera angle and framing.
    
- Facial structure and expression.
    
- Object placement.
    
- Background geometry.
    
- Lighting distribution.
    
- Interpretation of ambiguous prompt instructions.
    

These characteristics emerge during the denoising process.

The seed does not explicitly encode a pose, composition or camera position. Rather, different initial noise patterns can lead the model towards different solutions consistent with the conditioning.

This explains why two images generated with identical prompts and model settings can have substantially different compositions when their seeds differ.

## 6. Latent Dimensions and Noise Compatibility

The noise tensor has a defined shape.

Its dimensions depend on the model architecture, latent representation, requested image dimensions and batch configuration.

Changing the output resolution can change the required latent dimensions.

For example:

```
Seed: 123456
Resolution: 1024 x 1024
    |
    v
Noise Tensor A

Seed: 123456
Resolution: 768 x 1152
    |
    v
Noise Tensor B
```

Although the seed remains unchanged, the tensors have different shapes and should not be considered identical starting conditions.

Similarly, different model architectures may use incompatible latent representations.

Consequently, **a shared seed does not guarantee identical initial noise across arbitrary model architectures or resolutions**.

## 7. Practical Application in ComfyUI

Fixed seeds are particularly valuable when investigating changes to an established workflow.

### Controlled comparison

Consider a baseline generation:

```
Seed:       384726194
Checkpoint: Model A
Sampler:    Euler
Steps:      28
CFG:        4.0
LoRA:       Disabled
```

Generate an image, then change only CFG:

```
Seed:       384726194
Checkpoint: Model A
Sampler:    Euler
Steps:      28
CFG:        4.5
LoRA:       Disabled
```

Provided the noise-generation and execution conditions remain equivalent, the initial noise is preserved.

The difference between the images can therefore be investigated as the effect of the CFG change.

### Recommended experimental approach

1. Establish a baseline workflow and record its settings.
    
2. Fix the seed rather than randomising it.
    
3. Preserve the resolution, latent configuration and noise-generation method.
    
4. Change one parameter at a time.
    
5. Compare the resulting images.
    
6. Repeat with additional seeds when assessing general model behaviour.
    

The final step is important: an improvement observed with one seed may not generalise to other seeds.

A fixed seed is useful for isolating changes, but multiple seeds are needed to evaluate consistency.

## 8. Limitations and Caveats

### Identical seeds do not guarantee identical outputs

Reproducibility depends on the complete generation environment, not just the seed.

GPU operations, numerical precision, software updates and nondeterministic implementations can affect the result.

### Different architectures may not share equivalent noise

A seed can be reused numerically across architectures, but this does not mean the resulting noise tensors are identical or compatible.

### Noise may be introduced at additional stages

Some sampling algorithms and workflows introduce further stochastic noise during execution.

These operations may have their own random-number-generation behaviour.

Therefore, the initial noise is not necessarily the only source of randomness in a generation.

### Seed locking does not preserve composition

Keeping the seed fixed does not lock the composition when checkpoints, conditioning, LoRAs or sampling behaviour change.

It preserves the initial random conditions where the noise-generation setup is equivalent.

### A seed is not a universal image identifier

A seed cannot independently reproduce an image without the associated model, workflow, settings and execution context.

For reproducibility, preserve the complete workflow and relevant model versions.

## 9. Key Conclusions

1. **The seed generates the noise; it is not the noise itself.**
    
2. The same seed can reproduce the same initial noise when the generation conditions are equivalent.
    
3. The initial noise provides the starting conditions for diffusion sampling.
    
4. Checkpoints, LoRAs, conditioning, samplers, schedulers and guidance determine how that noise develops into an image.
    
5. Identical initial noise can produce very different images under different generation configurations.
    
6. Numerically adjacent seeds do not imply visually similar outputs.
    
7. Fixed seeds are essential tools for controlled workflow comparisons.
    
8. A seed alone does not guarantee reproducibility across resolutions, architectures, implementations or software environments.
    

The central principle is:

> **The seed establishes the starting conditions; the diffusion pipeline determines the journey and the final image.**