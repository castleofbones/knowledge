---

type: decision  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- krea-2
    
- lora
    
- model-architecture
    
- checkpoint-management
    
- image-generation
    

purpose: knowledge
---
## Decision summary

**Prefer a known, predictable base checkpoint with optional compatible LoRAs for targeted customisation, rather than routinely replacing the entire checkpoint to obtain specific character, detail or aesthetic characteristics.**

This is the preferred architecture for the existing modular ComfyUI image-generation workflows, particularly the Krea 2 workflow.

It is a default strategy, not an absolute restriction against specialised checkpoints.

A different checkpoint remains appropriate when it provides capabilities, model behaviour or quality that cannot reasonably be achieved with the existing foundation and compatible LoRAs.

## Context

The decision emerged from iterative Krea 2 Turbo experiments involving a university character with long pink hair.

An alternative checkpoint produced exaggerated hair length and volume, which proved difficult to correct through prompt refinement.

Returning to the stock Krea 2 Turbo checkpoint produced more restrained results closer to the intended reference.

The experiments suggested that the alternative checkpoint's learned preferences may have contributed to the unwanted styling.

Although that specific cause was not conclusively isolated, the experience highlighted a broader architectural issue:

**Replacing a checkpoint changes much more than the individual characteristic being targeted.**

The user also identified a practical benefit of a LoRA-oriented approach: reduced storage consumption compared with retaining numerous full checkpoint variants.

## 1. Why checkpoints are opinionated

A checkpoint contains the learned model parameters used during generation.

A fine-tuned checkpoint may alter the model's behaviour across many dimensions, including:

- Facial structure and proportions.
    
- Anatomy and body shapes.
    
- Hair styling and texture.
    
- Clothing interpretation.
    
- Lighting and contrast.
    
- Colour preferences.
    
- Artistic style.
    
- Prompt interpretation.
    
- Composition tendencies.
    

These influences are not necessarily independent.

A checkpoint selected for one desirable characteristic may also introduce unwanted behaviour elsewhere.

### Example from the Krea 2 experiments

The intended character had naturally dishevelled, hip-length pink hair.

Some generations using an alternative checkpoint produced highly exaggerated, flowing hair extending towards the knees or calves.

Further prompt refinement did not consistently resolve the issue.

The stock Krea 2 Turbo checkpoint subsequently produced results closer to the reference.

### Evidence classification

**Observed:** Different checkpoint configurations produced materially different visual interpretations.

**Hypothesis:** The alternative checkpoint's learned aesthetic preferences contributed to the exaggerated hairstyle.

**Not established:** The exact mechanism, the alternative checkpoint's identity, or whether the difference was exclusively caused by checkpoint selection.

The general architectural decision does not depend on treating that specific hypothesis as proven.

## 2. Why prefer a stable base checkpoint?

A consistent foundation improves the ability to understand and reproduce model behaviour.

Advantages include:

### Predictability

Prompt interpretation can be learned and evaluated against a relatively stable baseline.

### Diagnosability

When an output changes unexpectedly, fewer foundational variables need to be considered.

### Reproducibility

Experiments can record one base checkpoint and explicitly configured customisations.

### Workflow consistency

Existing prompt-conditioning, sampling, detailing and upscaling stages can remain structurally unchanged.

### Maintenance

Reducing unnecessary checkpoint proliferation can simplify model organisation and management.

The stock Krea 2 Turbo checkpoint performed well in the experiments documented here, making it a useful baseline for this particular workflow.

This does not establish it as the best checkpoint for every subject or style.

## 3. Why use LoRAs for targeted customisation?

A LoRA stores learned adaptations that can be applied to a compatible base model.

Rather than distributing another complete set of model weights, it represents a comparatively compact set of adjustments.

Depending on its training and architecture, a LoRA may be used to influence:

- Character identity.
    
- Artistic style.
    
- Clothing or costume details.
    
- Specific visual concepts.
    
- Particular rendering characteristics.
    

### Architectural benefit

The base model remains the foundation while individual influences can be introduced, adjusted or removed.

This makes LoRAs useful for modular experimentation.

For example, a character-specific LoRA can be tested at different strengths without replacing the entire checkpoint.

If the result introduces unwanted changes, its influence can be reduced or disabled.

### Important limitation

LoRAs are not perfectly isolated modifications.

A LoRA intended to influence one characteristic may also affect:

- Facial appearance.
    
- Anatomy.
    
- Clothing.
    
- Colour.
    
- Composition.
    
- General rendering style.
    

The extent of these effects depends on the LoRA, the model, its training and the applied strength.

LoRAs should therefore be treated as controllable influences, not guaranteed surgical adjustments.

## 4. Compatibility is mandatory

LoRAs are not universally interchangeable between image-generation architectures.

Their learned adjustments must be compatible with the structures and parameters they are intended to modify.

For example, an Illustrious or SDXL LoRA must not be assumed compatible with Krea 2.

Compatibility must be established for the actual model architecture and relevant loading implementation.

### Practical requirements

Before using a LoRA:

1. Identify the base model architecture.
    
2. Identify the architecture targeted by the LoRA.
    
3. Confirm compatibility using the LoRA's documentation or reliable model metadata.
    
4. Confirm that the installed loader supports the relevant model and LoRA format.
    
5. Test the LoRA with the intended checkpoint.
    
6. Evaluate its influence at controlled strengths.
    

A LoRA appearing in a model directory or being selectable by a loader does not, by itself, prove compatibility or correct behaviour.

### Version considerations

ComfyUI Core, ComfyUI Desktop, custom nodes and model-loading implementations may evolve independently.

Compatibility recommendations should be checked against the installed environment rather than assumed from unrelated installations or older documentation.

## 5. Storage efficiency

A full checkpoint typically occupies substantially more storage than a small or moderately sized LoRA.

The exact difference depends on model architecture, precision, quantisation and LoRA rank.

### Illustrative example

Consider ten checkpoint variants, each occupying 12 GB.

|Strategy|Illustrative storage|
|---|---|
|Ten complete checkpoints|120 GB|
|One 12 GB checkpoint plus ten 300 MB LoRAs|15 GB|
|Difference|105 GB|

These figures are hypothetical and do not represent measured sizes from the user's actual model collection.

They demonstrate the potential storage advantage when compatible LoRAs can replace otherwise redundant full checkpoint variants.

### Limitations

Not every checkpoint has an equivalent LoRA.

Some checkpoint fine-tunes introduce extensive changes that cannot be reproduced faithfully by a small LoRA.

Additional base architectures may still be necessary for fundamentally different capabilities.

The objective is therefore to eliminate unnecessary duplication, not to minimise the number of checkpoints at any cost.

## 6. Preferred workflow architecture

The existing ComfyUI workflow is modular, allowing the base generation path to operate independently of optional enhancements.

The preferred conceptual processing structure is:

```
Base checkpoint
       |
       v
Optional compatible LoRA stack
       |
       v
Model and conditioning preparation
       |
       v
Prompt conditioning
       |
       v
Sampling
       |
       v
VAE decoding
       |
       v
Optional refinement routes
       |
       +-- Eye/detail enhancement
       |
       +-- Manual inpainting
       |
       +-- Upscaling
       |
       v
Final output
```

This is a conceptual representation, not an exported ComfyUI workflow or an exact record of node connections.

The actual workflow uses established prompt-conditioning and optional processing routes. Its architecture should be preserved unless a demonstrated requirement justifies a change.

### LoRA handling

The established workflow uses rgthree's stacked LoRA-loading functionality, with a Multi-LoRA approach discussed as part of the architecture.

The precise final Multi-LoRA implementation and its compatibility with every intended model were not established in this discussion.

Therefore, the diagram represents the intended separation of concerns rather than a claim that every possible LoRA combination has been tested.

## 7. Clean baseline generation

The following configuration was confirmed during the Krea 2 Turbo experiments:

|Component|Value|
|---|---|
|Checkpoint|Stock Krea 2 Turbo|
|CFG|1.0|
|Steps|8|
|Seed|Fixed for comparisons|
|LoRAs|Disabled|
|Prompt enhancer|Disabled|
|Eye enhancement|Disabled|
|Inpainting|Disabled|

The sampler implementation and numerical seed were not captured in this discussion.

This configuration produced useful, detailed images without requiring additional processing.

### Architectural conclusion

The clean base path should remain independently usable.

Optional LoRAs and refinement stages should be enabled to meet a specific requirement rather than treated as mandatory parts of every generation.

## 8. Recommended evaluation procedure

When a new visual characteristic or style is required:

### Stage 1 — Evaluate the base model

Generate using the stock checkpoint without additional LoRAs.

Determine whether the desired result can be achieved through an accurate, appropriately detailed prompt.

### Stage 2 — Identify the remaining limitation

Distinguish between:

- Prompt-description errors.
    
- Seed-specific variation.
    
- Model interpretation preferences.
    
- Missing specialised knowledge.
    
- Localised rendering defects.
    
- Limitations of the underlying model.
    

Do not assume every discrepancy requires a different checkpoint.

### Stage 3 — Test a compatible LoRA

If a suitable LoRA exists, introduce it without simultaneously changing unrelated workflow components.

Record the LoRA identifier and strength.

### Stage 4 — Evaluate its effects

Compare the result against the clean baseline.

Assess both the intended improvement and any unwanted changes elsewhere.

### Stage 5 — Decide whether a checkpoint change is justified

Consider a different checkpoint when:

- The desired capability is absent from the existing foundation.
    
- Compatible LoRAs cannot reproduce the required result adequately.
    
- A specialised checkpoint demonstrates a meaningful quality advantage.
    
- The change is worth its storage, maintenance and reproducibility costs.
    

A checkpoint change should be an informed architectural choice rather than the automatic response to an unsatisfactory image.

## 9. Reproducibility requirements

For meaningful comparisons, preserve the following information:

|Parameter|Reason|
|---|---|
|Base checkpoint identifier|Identifies the model foundation|
|LoRA identifiers|Identifies applied adaptations|
|LoRA strengths|Controls their influence|
|Positive prompt|Records conditioning instructions|
|Negative prompt, if supported|Records exclusions|
|Seed|Controls the initial random input|
|Sampler and scheduler|Affect the generation process|
|Step count|Affects sampling|
|CFG|Affects guidance|
|Resolution|Affects composition and generation|
|Refinement stages|Identifies downstream modifications|

Not every model or workflow exposes every parameter.

For Krea 2 Turbo, the discussed workflow did not use negative prompt conditioning.

Record only settings actually supported and used by the relevant workflow.

## 10. Risks and trade-offs

### LoRA interaction

Multiple LoRAs can interact in ways that are difficult to predict.

Their effects are not necessarily additive or independent.

### Performance

LoRA loading and application have computational costs.

Switching LoRAs may be less expensive than loading entirely different checkpoints, but this depends on the implementation, caching and available resources.

No performance benchmark was conducted during this discussion.

### Quality differences

A specialised checkpoint may outperform a base checkpoint with LoRAs for particular tasks.

The preferred architecture should not prevent using such checkpoints when the benefit is demonstrated.

### Maintenance

LoRAs and custom loaders can introduce compatibility or version-management requirements.

The model foundation may remain stable while surrounding software changes.

### Reproducibility

A base checkpoint and LoRA combination must be recorded together.

Using the same prompt and seed with different LoRA strengths does not guarantee similar outputs.

## 11. Decision rationale

The decision balances four priorities.

**Predictability:** A familiar base checkpoint provides a more understandable foundation.

**Control:** Optional LoRAs allow specific learned influences to be introduced and adjusted.

**Efficiency:** Reusing a base checkpoint may substantially reduce storage requirements.

**Diagnosability:** Separating the foundation from customisations makes controlled experimentation easier.

The strategy is particularly well suited to the existing modular workflow, where prompt conditioning, model customisation and post-processing are already treated as separate concerns.

## 12. Outstanding work

The following work remains useful but was not completed or verified in this discussion:

- Inventory the existing checkpoint collection.
    
- Identify redundant or overlapping checkpoint variants.
    
- Record the exact alternative checkpoint used in the pink-hair experiments.
    
- Identify compatible LoRAs that could replace selected checkpoint variants.
    
- Measure actual storage savings.
    
- Validate the intended Multi-LoRA implementation against installed model loaders.
    
- Compare output quality and loading performance under controlled conditions.
    

These are future investigation items, not prerequisites for using the existing base workflow.

## Final decision

Use the stock Krea 2 Turbo checkpoint as the known baseline for the established Krea 2 workflow.

Prefer compatible, selectively applied LoRAs for targeted character, style and detail customisation where they provide suitable results.

Retain specialised checkpoints when they demonstrate capabilities or quality that the base model and LoRA approach cannot adequately reproduce.

Preserve a clean generation path without mandatory enhancements.

**The guiding principle is to keep the foundation stable and make specialised behaviour explicit, optional and testable.**

## Related reference

See `krea2-prompt-engineering-lessons.md` for the controlled image-generation experiments that informed this decision.