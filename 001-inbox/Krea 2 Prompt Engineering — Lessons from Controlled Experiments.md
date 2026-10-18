---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- krea-2
    
- prompt-engineering
    
- image-generation
    
- reproducibility
    

---
## Overview

This article documents practical findings from iterative image-generation experiments using Krea 2 Turbo in ComfyUI Desktop.

The experiments focused on reproducing a reference character: an adult university student with long pastel-pink hair, glasses, a white blouse and a black skirt, walking through a university corridor.

The investigation revealed important lessons about prompt accuracy, model interpretation, checkpoint behaviour, experimental controls and the risks of excessive prompt refinement.

The principal conclusion is that **prompt accuracy and a predictable model foundation are more valuable than increasing prompt complexity indiscriminately**.

## Environment and experimental configuration

The confirmed clean-generation configuration was:

|Component|Configuration|
|---|---|
|Application|ComfyUI Desktop, Windows 11|
|Model|Stock Krea 2 Turbo|
|CFG|1.0|
|Sampling steps|8|
|Seed|Fixed between controlled comparisons|
|LoRAs|None|
|Prompt enhancement|Disabled|
|Eye enhancement|Disabled|
|Inpainting|Disabled|
|Workflow|Existing modular base Krea 2 workflow|

The exact sampler implementation, seed value and checkpoint filename were not recorded in this discussion.

The experiments were conducted using the existing workflow's simplest generation path, without additional enhancement stages.

This distinction matters because it allowed changes in the generated images to be examined without interference from LoRAs, detailers or prompt rewriting.

## 1. Reference-image accuracy comes before prompt optimisation

### Observed problem

The original reference featured long, slightly dishevelled pink hair, with its longest visible sections reaching approximately hip level.

During prompt refinement, the hair was incorrectly described as extending towards the upper thighs.

One problematic instruction was:

> Extending past her waist and hips to reach her upper thighs, approximately halfway between her hips and knees.

This was an inaccurate description of the reference.

### Observed result

Successive generations produced increasingly exaggerated hair:

- Excessive volume around the crown.
    
- Large, separated sections falling down either side of the body.
    
- Long ribbon-like waves.
    
- Hair extending towards the knees or calves.
    
- A fantasy-inspired appearance inconsistent with the reference.
    

Additional descriptive language reinforced the exaggerated characteristic rather than correcting it.

### Lesson

Before refining a prompt, verify the visual characteristics of the reference itself.

In particular, distinguish:

- The longest visible extent of a feature.
    
- Its typical or predominant extent.
    
- Its volume and distribution.
    
- Its texture and physical behaviour.
    
- Its relationship to the rest of the subject.
    

A precise description of an incorrectly observed feature is still an incorrect prompt.

### Recommended practice

For hair, describe length using the correct anatomical endpoint and avoid repeatedly reinforcing the same characteristic.

A suitable description for the reference was:

> She has very long, dishevelled pastel rose-pink hair, falling loosely around her shoulders and down to her hips. Her hair is naturally tousled, with irregular loose waves, uneven layers and delicate wispy strands framing her face. It has soft volume around the crown and shoulders, with the longer sections gradually tapering towards her waist and hips.

This wording was proposed after identifying the reference-reading error. It should not be treated as a separately verified final prompt.

## 2. Excessive specificity can produce exaggerated results

### Observation

Several prompts repeatedly emphasised the same hair characteristics using expressions such as:

- Exceptionally long.
    
- Cascading generously.
    
- Extending past the waist and hips.
    
- Reaching the upper thighs.
    
- Exceptionally long lower sections.
    

The resulting images frequently exaggerated the length and volume.

### Interpretation

Repeated emphasis may increase the visual prominence of a feature in the model's interpretation.

However, the experiments do not establish a universal relationship between repeated wording and feature exaggeration. The checkpoint, seed and interactions between prompt terms may also contribute.

### Recommendation

Describe important characteristics clearly, but avoid repeating them throughout the prompt unless repetition has demonstrated value.

Prefer one coherent description that establishes:

1. The characteristic.
    
2. Its visible extent.
    
3. Its texture or construction.
    
4. Its relationship to surrounding features.
    

Additional wording should contribute new information rather than restating the same requirement.

## 3. Preserve successful prompt components

### Observed problem

An early revision attempted to correct the hair while simultaneously changing descriptions of:

- Facial structure.
    
- Skin tone.
    
- Body proportions.
    
- Clothing fit.
    
- Expression.
    
- Lighting.
    
- Photographic style.
    

The resulting character differed substantially from the preferred generation.

The hair was less exaggerated, but the character also became less recognisable.

### Lesson

**A prompt revision intended to fix one characteristic should not unnecessarily rewrite unrelated characteristics.**

Otherwise, it becomes difficult to determine which change caused the observed result.

### Recommended procedure

1. Identify the specific visual discrepancy.
    
2. Locate the prompt language most directly responsible.
    
3. Preserve the remaining prompt.
    
4. Change only the relevant description.
    
5. Reuse the seed and generation configuration.
    
6. Compare the results.
    
7. Keep or reject the change based on the observed effect.
    

This approach does not guarantee that unrelated image features remain identical. Diffusion models can produce substantial image changes from relatively small conditioning differences.

Nevertheless, limiting the changed variables improves the interpretability of the experiment.

## 4. Fixed seeds support controlled comparisons

### Confirmed experiment

Two generations were produced using:

- The same stock Krea 2 Turbo checkpoint.
    
- The same seed.
    
- CFG 1.0.
    
- Eight sampling steps.
    
- The same base workflow.
    
- No LoRAs or enhancement stages.
    

Only the positive prompts differed.

### Observed results

The generations showed differences in:

- Facial expression.
    
- Hair length and distribution.
    
- Body proportions.
    
- Clothing presentation.
    
- Overall character styling.
    

The broad university-corridor composition remained relatively consistent.

### Interpretation

A fixed seed preserves the initial random input for a given generation configuration.

It does not preserve character identity, pose, composition or appearance when conditioning changes.

### Practical rule

When investigating prompt behaviour, keep the seed fixed and modify one meaningful prompt variable at a time.

When investigating checkpoint behaviour, keep the prompt and other generation parameters unchanged where possible.

Record the actual seed and relevant model identifiers when preserving a reproducible experiment.

## 5. Checkpoint behaviour can dominate prompt interpretation

### Observed behaviour

An alternative checkpoint produced strongly stylised results, particularly exaggerated pink hair.

Subsequent experiments using the stock Krea 2 Turbo checkpoint produced more restrained hairstyles and images closer to the intended reference.

### Hypothesis

The alternative checkpoint may have contained learned aesthetic preferences that encouraged exaggerated hair length, volume or stylisation.

This would explain why increasingly explicit prompt descriptions failed to produce the desired result.

### Evidence limitations

The discussion did not establish:

- The exact identity of the alternative checkpoint.
    
- Its training characteristics.
    
- Whether every generation parameter was identical across checkpoint comparisons.
    
- Whether the behaviour would reproduce consistently across a larger sample.
    

Consequently, checkpoint bias is a plausible explanation, not a conclusively isolated cause.

### Lesson

When a model repeatedly interprets a characteristic in an unwanted way, consider whether the checkpoint itself is contributing to the result.

Do not assume every mismatch can or should be solved through additional prompt engineering.

## 6. Composition should be evaluated against the instructions

### Observation

The generated university portraits consistently placed the subject near the centre of the frame.

This was initially described as a possible weakness.

However, the prompts explicitly requested:

- A full-body portrait.
    
- A subject facing the camera.
    
- An eye-level photographic perspective.
    
- A clear central focal subject.
    
- A university corridor providing natural architectural perspective.
    

No rule-of-thirds placement or alternative framing was specified.

### Conclusion

The centred composition was a reasonable interpretation of the prompt, not evidence of a model failure.

The corridor's ceiling, windows, floor tiles and walls also provided natural leading lines.

### Lesson

Distinguish between:

- Failure to follow an explicit instruction.
    
- An acceptable interpretation of an unspecified characteristic.
    
- A creative preference that was never communicated.
    

A model should not be considered incorrect merely because it did not make an unstated creative decision.

## 7. Establish a clean generation before applying enhancements

### Confirmed result

The stock Krea 2 Turbo checkpoint produced visually strong images with:

- CFG 1.0.
    
- Eight steps.
    
- No LoRAs.
    
- No eye enhancement.
    
- No inpainting.
    
- No prompt enhancement.
    

This demonstrated that the base workflow could produce useful results without additional refinement stages.

### Architectural implication

Optional enhancements should remain optional.

The existing workflow separates base generation from additional processing, allowing enhancements to be introduced only when a specific need has been identified.

Examples include:

|Enhancement|Appropriate purpose|
|---|---|
|Eye detailer|Correcting demonstrated eye-detail problems|
|Inpainting|Localised corrections|
|Upscaling|Increasing output resolution or detail|
|LoRAs|Introducing compatible, specialised learned influences|
|Prompt enhancer|Experimenting with prompt expansion or refinement|

These are possible uses, not evidence that each enhancement was required or tested successfully during this experiment.

### Recommendation

Always evaluate the base generation before enabling additional stages.

Avoid introducing multiple enhancement mechanisms simultaneously when investigating a problem.

## 8. Prompt enhancement requires its own baseline

The user's prompt enhancer was deliberately disabled during the controlled experiments.

This ensured that the submitted positive prompt was the conditioning text being evaluated, without an additional rewriting stage.

The experiments therefore establish a useful baseline for future comparisons between:

- Manually authored prompts.
    
- Automatically enhanced prompts.
    
- Alternative prompt-construction methods.
    

No controlled conclusion about the enhancer's quality or reliability was established in this discussion.

## 9. Photographic realism and reference fidelity are different objectives

The experiments also highlighted a distinction between a visually attractive generation and a faithful reproduction of the reference.

A generated image may have excellent:

- Lighting.
    
- Skin rendering.
    
- Fabric detail.
    
- Facial attractiveness.
    
- Background separation.
    
- Photographic composition.
    

Yet still differ from the intended character in hair, expression, body proportions or clothing.

Similarly, a reference may contain stylised facial proportions that a photorealistic model interprets differently.

### Recommendation

Define the objective before evaluating a generation.

For reference reproduction, prioritise fidelity to the reference's distinctive features.

For creative reinterpretation, allow deliberate deviations while retaining the intended character and atmosphere.

Do not treat photographic realism as a substitute for character accuracy.

## 10. Practical experimental method

For future Krea 2 prompt investigations:

1. Start with a known checkpoint and the simplest available workflow path.
    
2. Record the checkpoint identity, prompt, seed and generation parameters.
    
3. Inspect the reference before modifying the prompt.
    
4. Identify the largest meaningful discrepancy.
    
5. Change one relevant description.
    
6. Generate using the same seed.
    
7. Compare both the intended change and any unintended changes.
    
8. Preserve successful configurations before experimenting further.
    
9. Test checkpoint or LoRA influences separately when appropriate.
    
10. Introduce detailers, inpainting and upscaling only when justified by the output.
    

For experiments involving stochastic variation, additional seeds may help distinguish a recurring interpretation from a seed-specific result.

## Conclusions

The experiments support several durable principles:

- Accurate observation is a prerequisite for accurate prompting.
    
- Longer prompts are not necessarily better prompts.
    
- Repeated emphasis can contribute to unwanted exaggeration.
    
- Preserve successful prompt components during targeted revisions.
    
- Fixed seeds improve experimental control but do not lock composition.
    
- Checkpoint selection can materially affect prompt interpretation.
    
- Evaluate the model against instructions actually supplied.
    
- A clean base workflow is valuable for diagnosis and reproducibility.
    
- Enhancements should solve identified problems rather than become mandatory processing stages.
    

The most important principle is:

**Treat prompt engineering as controlled experimentation, not an endless process of adding descriptive language.**

## Related architectural decision

The companion article, `checkpoint-vs-lora-strategy.md`, documents the decision to favour a predictable base checkpoint and compatible, selectively applied LoRAs over frequent replacement of the entire model foundation.