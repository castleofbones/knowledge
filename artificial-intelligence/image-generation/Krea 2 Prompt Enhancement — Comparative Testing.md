---

type: experiment  
status: complete  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- krea-2
    
- qwen3-vl
    
- prompt-enhancement
    
- danbooru
    
- comparative-testing
    
- image-generation
    

purpose: knowledge
---
## Overview

This experiment evaluated whether Qwen3-VL prompt enhancement improves image-generation results when using Krea 2 Turbo in ComfyUI.

The comparison examined two fundamentally different input formats:

1. A carefully authored natural-language prompt.
    
2. A sparse, comma-separated Danbooru-style prompt.
    

Each input was tested both directly and after Qwen enhancement.

An additional generation from the original, unmodified workflow provided a fifth baseline image.

The experiment was designed to distinguish improvements in **prompt interpretation and semantic coherence** from general differences in image quality.

> [!important] Principal finding  
> Prompt enhancement provided the clearest benefit when converting Danbooru tags into coherent natural language. It did not demonstrate a consistent improvement over carefully authored prose.

## 1. Objectives

The experiment sought to answer four questions.

**Does Krea 2 require natural-language prompts?**

Determine whether Krea 2 can interpret Danbooru tags directly without an intermediate translation stage.

**Does Qwen enhancement improve tag interpretation?**

Determine whether converting tags into prose produces more coherent relationships between subjects, objects, poses and environments.

**Does enhancement improve detailed prompts?**

Determine whether Qwen provides a measurable benefit when the original prompt is already well structured.

**Does enhancement preserve semantic fidelity?**

Identify whether the enhancer changes important characteristics, invents relationships or alters composition.

The objective was not simply to determine which image looked most attractive.

## 2. Test Environment

The experiment used the existing ComfyUI Desktop Krea 2 workflow.

### Relevant configuration

|Component|Configuration|
|---|---|
|Operating system|Windows 11|
|GPU|NVIDIA RTX 5070 Ti, 16 GB|
|ComfyUI|Desktop|
|ComfyUI Core|v0.40.0 historical baseline|
|Image model|`krea2_turbo_fp8_scaled.safetensors`|
|Text model|`qwen3vl_4b_fp8_scaled.safetensors`|
|VAE|`qwen_image_vae.safetensors`|
|Prompt enhancer|Qwen3-VL|
|System prompt|Custom v1.3|
|Image-generation steps|8|
|CFG|1.0|
|Resolution|1344 × 2016|
|Seed|Fixed|

The generation settings represent the established Krea 2 Turbo baseline. The original workflow configuration should be retained alongside the images for exact reproducibility.

The comparison used the same generation path and fixed seed while changing the selected prompt.

### Experimental controls

The intended controlled variables were:

- Image-generation model
    
- Text encoder
    
- VAE
    
- Image dimensions
    
- Sampling configuration
    
- CFG
    
- Seed
    
- Downstream image-generation workflow
    

The principal independent variables were:

- Input prompt format
    
- Whether Qwen enhancement was enabled
    

The experiment did not attempt to evaluate multiple seeds or quantify statistical variation.

Consequently, the findings describe the observed results of this controlled comparison rather than a general performance benchmark.

## 3. Experimental Design

The experiment used two independent prompt-routing switches.

### Switch A — Prompt source

Selected between:

- **Good:** Carefully written natural-language description.
    
- **Shit:** Raw Danbooru-style tags.
    

The informal names were used during development to make the routes immediately distinguishable.

They do not imply that Danbooru tags are inherently inferior.

### Switch B — Enhancement

Selected between:

- **CUI:** Original prompt passed directly to conditioning.
    
- **ENH:** Prompt processed through Qwen3-VL before conditioning.
    

This produced four combinations.

|Test|Prompt source|Enhancement|
|---|---|---|
|CUI User|Detailed prose|Off|
|ENH User|Detailed prose|On|
|CUI Shit|Danbooru tags|Off|
|ENH Shit|Danbooru tags|On|

A fifth output, **Normal**, was generated using the original workflow configuration.

### Conceptual architecture

```
Detailed User Prompt ──┐
                       ├── Source Switch ────┬── Raw Prompt ─────┐
Danbooru Tags ─────────┘                     │                   │
                                             └── Qwen Enhancer ──┤
	                                                             │
                                                         Enhancement Switch
                                                                 │
                                                                 v
                                                          Selected Prompt
                                                                 │
                                                                 v
                                                         CLIP Text Encode
                                                                 │
                                                                 v
                                                            CONDITIONING
                                                                 │
                                                                 v
                                                         Krea 2 Generation
```

The two switches allowed source format and enhancement to be varied independently.

Both paths converged before CLIP text encoding.

The enhancement system prompt was used only by Qwen and was not included in the final Krea 2 conditioning text.

## 4. Test Inputs

### Danbooru input

The following tag list describes an adult woman in a dimly lit café or bar environment.

```
1girl, solo, adult woman, platinum blonde hair, long hair, wavy hair,
green eyes, charcoal turtleneck, black pleated skirt, brown pantyhose,
black knee-high boots, jewelry, coffee cup, sitting, crossed legs,
looking at viewer, cafe, bar, wine bottles, dim lighting,
depth of field, bokeh, photorealistic
```

The tags specify:

- A single adult female subject.
    
- Platinum-blonde, long, wavy hair and green eyes.
    
- A charcoal turtleneck and black pleated skirt.
    
- Brown pantyhose and black knee-high boots.
    
- Jewellery and a coffee cup.
    
- A seated pose with one leg crossed over the other.
    
- Direct eye contact.
    
- A café/bar environment with wine bottles.
    
- Dim lighting, depth of field and bokeh.
    
- A photorealistic photographic interpretation.
    

The tag list does not explicitly specify all spatial relationships between these elements.

For example, the presence of a bar does not mean the subject is seated at the bar counter.

### Detailed natural-language input

The detailed prose prompt described the same general subject and setting using connected natural language.

Its purpose was to provide a stronger semantic starting point than the fragmented tag list.

The exact original prose prompt is not preserved in this note.

> [!warning] Reproducibility limitation  
> The exact detailed user prompt and all five generated image files should be retained with the original workflow export. Without those artefacts, the experiment can be understood and its conclusions preserved, but cannot be reproduced exactly from this note alone.

## 5. Prompt Enhancement Configuration

The enhancement path used Qwen3-VL with the custom v1.3 system prompt.

The system prompt had been developed specifically to reduce semantic drift.

Its principal requirements were:

- Preserve subjects, clothing, colours and physical characteristics.
    
- Preserve poses and spatial relationships.
    
- Convert tags into natural language.
    
- Avoid inventing scene content.
    
- Avoid arbitrary framing and cropping.
    
- Describe requested photorealism as a photograph.
    
- Add restrained photographic detail where appropriate.
    
- Produce only the final prompt.
    

### Generate Text settings

|Parameter|Value|
|---|---|
|Maximum length|512|
|Sampling|Enabled|
|Temperature|0.70|
|Top-K|64|
|Top-P|0.95|
|Min-P|0.05|
|Repetition penalty|1.05|
|Seed|0|
|Presence penalty|0.00|

The settings were retained from the existing template configuration.

The experiment was not designed to optimise Qwen's sampling parameters.

For the complete tested system prompt and its development history, see [[Qwen3-VL Prompt Enhancement for Krea 2]].

## 6. Generated Outputs

Five images were compared.

|Output identifier|Description|
|---|---|
|`workflow-cui-user`|Detailed prose, no enhancement|
|`workflow-enh-user`|Detailed prose, Qwen enhancement|
|`workflow-cui-shit`|Raw Danbooru tags|
|`workflow-enh-shit`|Danbooru tags enhanced by Qwen|
|`workflow_normal`|Original workflow baseline|

The identifiers reflect the names used during testing.

The original image files should be linked or embedded here when available.

### Suggested attachment structure

```
attachments/
└── krea2-prompt-comparison/
    ├── workflow-cui-user.png
    ├── workflow-enh-user.png
    ├── workflow-cui-shit.png
    ├── workflow-enh-shit.png
    └── workflow_normal.png
```

These are suggested attachment names, not verified original filenames.

## 7. Observations

### Test A — Detailed prose without enhancement

**Output:** `workflow-cui-user`

The original detailed prompt produced a strong photographic result.

The subject, clothing and overall café environment were interpreted convincingly.

The result demonstrated that Krea 2 was capable of producing a high-quality image directly from a carefully authored prompt.

**Finding:** Enhancement was not necessary to achieve a convincing photographic interpretation.

### Test B — Detailed prose with enhancement

**Output:** `workflow-enh-user`

The enhanced prompt also produced a convincing photograph.

However, Qwen's rewriting changed the interpretation of the original description.

The result exhibited a somewhat different fashion/editorial presentation rather than an unambiguous improvement in photographic quality.

The enhancer was not merely cleaning up the text; it was influencing the generated scene.

**Finding:** Enhancing detailed prose can introduce variation without necessarily improving fidelity or image quality.

### Test C — Raw Danbooru tags

**Output:** `workflow-cui-shit`

Krea 2 interpreted the raw tag list surprisingly well.

The generated image contained a convincing subject and photographic environment.

This established that Danbooru-style tags were not inherently unsuitable for Krea 2.

However, the result was comparatively generic in its interpretation of some details.

The relationship between jewellery, coffee cup, subject pose and environmental elements was less clearly expressed.

**Finding:** Krea 2 can generate strong photographic images directly from Danbooru tags, but some scene relationships may remain weak or ambiguous.

### Test D — Enhanced Danbooru tags

**Output:** `workflow-enh-shit`

The Qwen-enhanced Danbooru prompt produced a more cohesive scene.

The photographic interpretation showed stronger integration of the subject, café/bar setting, accessories and coffee-related action.

The improvement was primarily one of semantic coherence and scene organisation.

It was not simply a matter of sharper detail or greater photorealism.

**Finding:** Converting tags into natural language can improve how Krea 2 interprets relationships between scene elements.

### Test E — Original workflow baseline

**Output:** `workflow_normal`

The original, unmodified workflow also produced a strong result.

This provided an important reference against which to evaluate the experimental routes.

It demonstrated that the existing Krea 2 workflow was already capable of high-quality photographic generation.

**Finding:** Prompt enhancement should not be credited for improvements that may already be achievable through the baseline workflow.

## 8. Comparative Results

|Evaluation criterion|Raw detailed prose|Enhanced detailed prose|Raw Danbooru|Enhanced Danbooru|
|---|---|---|---|---|
|Convincing photography|Strong|Strong|Strong|Strong|
|Subject interpretation|Strong|Strong|Good|Good–strong|
|Environmental coherence|Strong|Strong|Less coherent|Improved|
|Accessory integration|Good|Good|Less consistent|Improved|
|Preservation of original wording|Direct|Rewritten|Direct|Rewritten|
|Risk of semantic reinterpretation|Lower|Higher|Lower from enhancement|Higher from enhancement|
|Clear benefit from enhancement|Not applicable|Not demonstrated|Not applicable|Observed|

These are qualitative assessments of the particular generated images.

They are not numerical scores or statistical measurements.

### Main distinction

The raw Danbooru output was not a failed generation.

It was already visually convincing.

The enhanced version improved the interpretation of the intended scene rather than rescuing an unusable result.

That distinction is important when evaluating the actual value of the enhancement component.

## 9. Prompt Fidelity Versus Image Quality

A visually attractive image is not necessarily a faithful interpretation of the input.

Similarly, a prompt that is semantically accurate does not guarantee the most aesthetically pleasing output.

The experiment therefore requires two separate evaluation dimensions.

### Prompt fidelity

Questions include:

- Are the requested subjects present?
    
- Are clothing and colours preserved?
    
- Are poses correctly interpreted?
    
- Are specified objects visible?
    
- Are spatial relationships preserved?
    
- Has anything important been invented or omitted?
    

### Image quality

Questions include:

- Does the image look photographically convincing?
    
- Are materials and textures believable?
    
- Is the lighting coherent?
    
- Is the composition effective?
    
- Are facial features and anatomy plausible?
    
- Is the overall image aesthetically successful?
    

Qwen enhancement primarily operates on the first dimension.

Its effect on the second dimension is indirect, mediated through Krea 2's interpretation of the rewritten prompt.

### Example: pose terminology

The distinction between:

```
sitting, crossed legs
```

and:

```
sitting cross-legged
```

illustrates the problem.

Both descriptions can generate aesthetically convincing images.

However, they describe materially different poses.

A visually attractive result using the wrong pose would still represent a semantic failure.

## 10. Interpretation of Results

The experiment supports three principal conclusions.

### 10.1 Krea 2 does not require prompt enhancement

The model produced strong results from both detailed prose and raw Danbooru tags.

Therefore, enhancement should not be treated as a mandatory processing stage.

### 10.2 Enhancement is most useful for fragmented inputs

Danbooru tags describe concepts but often omit grammatical relationships.

Qwen can provide those relationships through natural-language reconstruction.

This can improve the coherence of the resulting image.

However, the reconstruction may also introduce assumptions not present in the original tags.

### 10.3 Detailed prompts benefit less consistently

A carefully authored prompt already contains explicit scene relationships.

Rewriting it may alter emphasis, framing or stylistic interpretation.

The enhanced output may be different without being objectively better.

This argues against enabling enhancement automatically for every input.

## 11. Architectural Consequences

The results influenced the design of the planned Prompt Conditioning subgraph.

### Enhancement must remain optional

The workflow should preserve both original and enhanced text paths.

```
Selected Prompt
      |
      +───────────────+
      |               |
      v               v
  Raw Prompt      Qwen Enhanced
      |               |
      +───────┬───────+
              |
              v
      Enhancement Switch
              |
              v
       Final Prompt Text
              |
              v
       CLIP Text Encode
              |
              v
         CONDITIONING
```

### Final prompt text should be exposed

The selected text should remain accessible independently of conditioning.

This supports:

- Inspection of Qwen output.
    
- Copying prompts into other workflows.
    
- Manual refinement.
    
- Repeated experimentation.
    
- Comparing original and enhanced descriptions.
    
- Building image-to-prompt-to-image pipelines.
    

### Enhancement should precede CLIP encoding

Switching between raw and enhanced text before encoding allows both paths to share the same CLIP Text Encode node.

It also makes the prompt used for generation easier to inspect.

### Preserve the original input

Enhancement should never irreversibly replace the original prompt.

The original remains the reference for evaluating semantic fidelity.

See [[ComfyUI — Prompt Conditioning — Subgraph Design]].

## 12. Limitations

### Single-scene experiment

The test used one principal scene involving an adult woman in a café/bar environment.

Results may differ for landscapes, architecture, fantasy characters, multiple subjects or complex action scenes.

### Single fixed seed

The comparison controlled the generation seed but did not evaluate multiple seeds.

It therefore cannot establish how consistently enhancement improves results across the model's output distribution.

### Qualitative evaluation

The images were evaluated visually.

No automated image-quality, prompt-alignment or semantic-fidelity metrics were collected.

### Qwen sampling variability

The prompt-enhancement model used sampling.

The experiment did not independently quantify variation between repeated enhancement runs.

### Missing exact prose input

The complete original detailed prose prompt is not included in this note.

Exact reproduction requires the original workflow or saved prompt text.

### Model and workflow specificity

The findings apply to the tested Krea 2 Turbo workflow and Qwen3-VL configuration.

They should not automatically be generalised to Illustrious, Flux, Anima or other architectures.

## 13. Recommended Operating Strategy

Based on the observed results:

|Input type|Default enhancement|
|---|---|
|Carefully authored detailed prose|Off|
|Sparse natural-language description|On, when useful|
|Raw Danbooru tags|On|
|Mixed prose and tags|Evaluate individually|
|Prompts requiring exact fidelity|Off initially; compare before accepting enhancement|
|Creative exploration|Optional|

These are practical defaults, not hard rules.

The final choice should remain under user control.

### Future validation

A broader experiment could evaluate:

- Multiple seeds per prompt.
    
- Several unrelated scenes.
    
- Character and environmental complexity.
    
- Exact pose preservation.
    
- Object omission and invention rates.
    
- Multiple Qwen sampling temperatures.
    
- Different image-generation models.
    

This would help determine whether the observed benefit is consistent beyond the original test scene.

## 14. Lessons Learned

1. **Do not assume that more processing produces better results.**
    
2. Raw Danbooru tags can work surprisingly well with Krea 2.
    
3. Natural-language enhancement can improve semantic relationships in fragmented input.
    
4. Detailed prose may already provide everything the image model needs.
    
5. Enhancement can introduce interpretation changes even when instructed to preserve meaning.
    
6. Image quality and prompt fidelity are separate evaluation criteria.
    
7. Fixed-seed comparisons are useful but insufficient for statistical conclusions.
    
8. A strong baseline is essential for assessing the value of additional processing.
    
9. Optional routing is preferable to mandatory enhancement.
    
10. Intermediate text outputs are important for debugging and reproducibility.
    

## Related Notes

- [[ComfyUI — Workflow Architecture & Design Patterns]]
    
- [[Qwen3-VL Prompt Enhancement for Krea 2]]
    
- [[ComfyUI — Prompt Conditioning — Subgraph Design]]
    
- [[ComfyUI — Hybrid WD14 + Qwen Image-to-Illustrious Prompt Pipeline]]
    

## Conclusion

The experiment demonstrated that Qwen3-VL prompt enhancement can provide a meaningful benefit when translating Danbooru-style tags into natural-language descriptions for Krea 2 Turbo.

The benefit was most apparent in the coherence of scene relationships rather than in a universal improvement in photographic quality.

Conversely, enhancing carefully authored prose produced a different interpretation without demonstrating a clear advantage over the original prompt.

The resulting architectural decision is to retain **optional prompt enhancement**, preserve the original input and expose the final selected text before conditioning.

The broader lesson is that prompt enhancement should solve a specific problem, not become an unquestioned part of every image-generation workflow.