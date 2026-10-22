---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- krea-2
    
- qwen3-vl
    
- prompt-engineering
    
- prompt-enhancement
    
- danbooru
    
- image-generation
    

purpose: knowledge
---
## Overview

This article documents the development and testing of a Qwen3-VL-based prompt-enhancement component within a ComfyUI Krea 2 image-generation workflow.

The component transforms natural-language descriptions, Danbooru tags or mixtures of both into coherent, descriptive prompts suitable for Krea 2.

The primary objective is **faithful semantic translation**, not unrestricted creative expansion.

The most useful application discovered during testing was converting sparse Danbooru-style input into natural-language descriptions while preserving the original subject, clothing, pose, environment and visual intent.

> [!important] Principal finding  
> Qwen3-VL prompt enhancement is most valuable as a **semantic translator and prompt normaliser**, rather than an automatic quality improvement applied indiscriminately to already detailed prompts.

## 1. Environment and Model

The implementation was developed using ComfyUI Desktop on Windows 11.

### Relevant environment

|Component|Configuration|
|---|---|
|ComfyUI|Desktop installation|
|ComfyUI Core|v0.40.0 at time of testing|
|GPU|NVIDIA RTX 5070 Ti, 16 GB VRAM|
|Text model|`qwen3vl_4b_fp8_scaled.safetensors`|
|Image model|`krea2_turbo_fp8_scaled.safetensors`|
|VAE|`qwen_image_vae.safetensors`|
|Prompt-generation node|`Generate Text`|
|Prompt-enhancement system prompt|Custom v1.3|

These versions and filenames describe the tested configuration, not requirements for future installations.

### Model reuse

The prompt-enhancement component reuses the Qwen text-model connection already present in the Krea 2 workflow.

It does not require loading a second copy of Qwen solely for enhancement.

This reduces unnecessary model-loading overhead and keeps the component integrated with the existing workflow.

## 2. Purpose and Scope

Krea 2 accepts natural-language prompts and can interpret Danbooru-style tags directly.

However, tag lists often lack explicit relationships between concepts.

For example:

```
1girl, solo, adult woman, platinum blonde hair,
long hair, wavy hair, green eyes,
charcoal turtleneck, black pleated skirt,
brown pantyhose, black knee-high boots,
jewelry, coffee cup, sitting, crossed legs,
looking at viewer, cafe, bar, wine bottles,
dim lighting, depth of field, bokeh,
photorealistic
```

The intended enhancement is a coherent description that preserves the information contained in the tags.

The enhancer must not introduce arbitrary scene elements, alter poses or invent composition that conflicts with the requested subject.

### Supported input types

- Detailed natural-language prompts
    
- Sparse natural-language prompts
    
- Danbooru-style tags
    
- Mixed prose and tag input
    

### Desired output

A single, cohesive natural-language image prompt that can be passed directly to Krea 2's text-conditioning stage.

The output should not include reasoning, commentary, Markdown, headings or explanatory text.

## 3. Workflow Architecture

The official ComfyUI Krea 2 template contains prompt enhancement within its Image Generation subgraph.

For development and experimentation, the enhancement nodes were instead exposed on the main canvas.

This allowed direct inspection of inputs, outputs and switching behaviour.

### Conceptual processing path

```
User / Danbooru Prompt
          |
          +------------------------+
          |                        |
          v                        |
     Generate Text                 |
          |                        |
          v                        |
   Enhanced Prompt                 |
          |                        |
          +----------+-------------+
                     |
                     v
              Raw / Enhanced
                   Switch
                     |
                     v
              Selected Prompt
                     |
                     v
              CLIP Text Encode
                     |
                     v
                CONDITIONING
                     |
                     v
              Krea 2 Generation
```

The enhancement system prompt is supplied to Qwen's text-generation operation.

It is **not** itself passed into Krea 2's image-conditioning prompt.

The original and enhanced prompt paths converge before CLIP encoding.

This permits both routes to use the same downstream conditioning operation.

### Intended component interface

The planned Prompt Conditioning subgraph exposes:

- Selected final positive-prompt text
    
- Positive conditioning
    

Exposing the text is important because it allows the generated prompt to be copied, inspected, edited or reused in subsequent workflows.

See [[ComfyUI — Prompt Conditioning — Subgraph Design]].

## 4. Generate Text Settings

The following settings were observed in the official ComfyUI Krea 2 template and used as the reference configuration for our experiments.

|Parameter|Value|
|---|---|
|`max_length`|512|
|Sampling Mode|Enabled|
|Temperature|0.70|
|Top-K|64|
|Top-P|0.95|
|Min-P|0.05|
|Repetition Penalty|1.05|
|Seed|0|
|Presence Penalty|0.00|

The node also exposes a thinking option.

Thinking had previously been important for reliable output in related Qwen workflows, but its precise effect on this enhancement experiment was not independently isolated.

### Sampling considerations

Temperature 0.70 allows variation in the generated text.

Consequently, identical inputs may not always produce identical outputs, depending on the node's seed handling and sampling implementation.

A lower temperature, such as 0.3–0.4, was identified as a possible future experiment for improving translation consistency.

**This was not established as a superior setting during the documented tests.**

The final system prompt was therefore retained without changing the sampling configuration.

## 5. System Prompt Development

The enhancer went through several iterations.

The experiments revealed that a longer, more prescriptive system prompt did not necessarily improve compliance.

### Initial official system prompt

The official template's system prompt instructed Qwen to expand a user's description into a more effective text-to-image prompt.

Its priorities included:

- Faithfulness to the input
    
- Practical image-generation structure
    
- Internal planning of style and composition
    
- Avoiding unnecessary changes to detailed prompts
    
- Producing a single final paragraph
    

This was a reasonable starting point but proved too permissive for some tag-based inputs.

### Detailed natural-language input

A detailed photographic prompt describing a woman in a café was submitted.

The output largely preserved the original description, removed formatting and combined the content into a paragraph.

Only minor refinements were introduced.

**Finding:** The official prompt behaved primarily as a conservative editor when the source description was already detailed.

### Sparse natural-language input

A shorter version of the same scene was tested.

Qwen added modest atmospheric descriptions such as soft shadows and bokeh highlights, but little substantial photographic enrichment.

**Finding:** The official enhancer was conservative even when additional descriptive detail would have been useful.

### Danbooru input

The tag-based version of the scene produced more revealing results.

Qwen successfully converted the tags into readable prose, but introduced several problems:

- Explanatory text appeared before the finished prompt.
    
- A medium or waist-up framing was introduced despite lower-body clothing being specified.
    
- Some environmental and lighting details were reinterpreted.
    
- Pose terminology was not reliably preserved.
    

These failures established the need for stricter semantic-preservation rules.

## 6. Failed Long-Form Custom Prompt

An initial custom system prompt attempted to address every observed failure through extensive rules.

It covered:

- Subject and scene fidelity
    
- Clothing and material preservation
    
- Pose semantics
    
- Composition
    
- Photography
    
- Tag interpretation
    
- Human anatomy
    
- Output formatting
    

However, this long prompt caused Qwen to repeat or echo instructions rather than reliably produce the requested image prompt.

Output could truncate while reproducing the system prompt.

Adding an explicit input delimiter did not resolve the behaviour.

### Interpretation

The failure was consistent with the prompt being too demanding or poorly handled by this particular Qwen 4B and Generate Text configuration.

However, the precise cause was not conclusively isolated.

The `max_length` setting, prompt structure and model behaviour were all plausible contributing factors.

**Lesson:** Increasing system-prompt length and specificity is not guaranteed to improve instruction compliance.

A compact prompt proved more effective.

## 7. Compact Prompt and Semantic Corrections

A shorter system prompt immediately improved results.

It successfully:

- Converted `1girl, solo` into a single adult woman.
    
- Preserved the main clothing and appearance details.
    
- Removed unwanted explanatory paragraphs.
    
- Avoided inventing a waist-up composition.
    
- Produced a cohesive natural-language description.
    

However, subsequent testing exposed several remaining problems.

### Problem: crossed legs

Input:

```
sitting, crossed legs
```

Incorrect output:

```
sitting cross-legged
```

These are different poses.

The intended meaning was one leg crossed over the other, not a cross-legged or lotus-style seated position.

The system prompt was updated with an explicit example and prohibition.

### Problem: café and bar

Input:

```
cafe, bar, wine bottles
```

One output interpreted this as:

```
cafe or bar
```

Another placed the subject at a café bar counter.

Neither relationship was specified by the input.

The system prompt was updated to preserve compatible environmental details without inventing spatial relationships.

### Problem: photographic medium

The model sometimes used:

```
photorealistic image
```

or:

```
rendered in photorealistic style
```

These were considered less appropriate than explicitly requesting a photograph.

The system prompt was updated to favour:

```
photorealistic photograph
```

### Problem: insufficient photographic enrichment

Sparse prompts were often converted into prose without much additional useful detail.

The prompt was therefore allowed to add restrained photographic descriptions relating to already specified elements, such as:

- Natural skin texture
    
- Realistic hair strands
    
- Fabric and material texture
    
- Believable lighting
    
- Depth and focus
    

This permission was constrained to avoid inventing new scene content.

## 8. Final Tested System Prompt — v1.3

The following is the complete system prompt used in the successful final text-generation test.

It is preserved verbatim for reproducibility.

```
You are a prompt editor for text-to-image generation. Convert the USER INPUT into one clear, cohesive natural-language image prompt.

The input may be prose, comma-separated tags, Danbooru tags, or a mixture.

Rules:
- Preserve all meaningful subjects, appearance, age, body features, clothing, colors, materials, objects, actions, poses, expressions, environment, lighting, composition, spatial relationships, and style.
- Do not invent new people, objects, clothing, accessories, colors, scenery, narrative events, locations, or spatial relationships.
- Translate tag syntax into fluent natural language rather than repeating tag vocabulary mechanically. For example, "1girl, solo, adult woman" becomes "a single adult woman".
- Do not reinterpret ambiguous tags by inventing relationships between them. Preserve compatible scene tags independently unless their relationship is explicitly stated. For example, "cafe, bar, wine bottles" may become "a cafe with a bar and wine bottles visible in the background"; it does NOT mean the subject is sitting at a bar counter.
- Preserve pose terminology precisely. "sitting, crossed legs" means the subject is sitting with one leg crossed over the other. NEVER translate "crossed legs" as "cross-legged", "sitting cross-legged", lotus position, or legs folded beneath the subject unless explicitly requested.
- Never invent framing, crop, camera angle, viewpoint, or composition that could hide requested details. If footwear, legs, or lower-body clothing are specified, keep them compatible with the visible composition.
- Preserve the requested medium, mood, lighting, realism, and visual intent.
- If "photorealistic" or photography is requested, describe the result as a "photorealistic photograph", not a "photorealistic image", "photorealistic style", illustration, render, or painting.
- For sparse photorealistic input, add restrained rendering detail that improves photographic realism without changing scene content. You may describe natural skin texture, realistic hair strands, believable fabric and material texture, natural light behavior, dimensional form, realistic focus, depth of field, and photographic detail when relevant to elements already present.
- Do not add specific physical characteristics, clothing construction, accessories, colors, objects, or environmental details that were not supplied.
- For already detailed input, lightly refine rather than unnecessarily expanding it.
- Prefer concrete visible descriptions over metaphor, storytelling, emotional interpretation, or flowery prose.
- Do not invent camera models, lenses, focal lengths, apertures, film stocks, photographers, or photographic techniques unless explicitly requested.
- Output ONLY the finished image-generation prompt. No reasoning, explanation, summary, headings, labels, markdown, or introductory text.

USER INPUT:
```

The actual user description or Danbooru tags are appended after `USER INPUT:`.

The final delimiter is part of the system-prompt text.

## 9. Final Validation Test

### Input

```
1girl, solo, adult woman, platinum blonde hair, long hair, wavy hair,
green eyes, charcoal turtleneck, black pleated skirt, brown pantyhose,
black knee-high boots, jewelry, coffee cup, sitting, crossed legs,
looking at viewer, cafe, bar, wine bottles, dim lighting,
depth of field, bokeh, photorealistic
```

### Generated output

```
A photorealistic photograph of a single adult woman sitting at a cafe table with crossed legs, looking directly at the viewer. She has platinum blonde wavy long hair and green eyes, wearing a charcoal turtleneck, a black pleated skirt, brown pantyhose, and black knee-high boots. She is adorned with jewelry and holds a coffee cup in her hand. The setting includes a bar in the background with wine bottles visible, illuminated by dim lighting. The image features depth of field and bokeh, with soft focus on the background elements while the subject remains sharply defined.
```

### Evaluation

|Requirement|Result|
|---|---|
|Single adult woman|Preserved|
|Platinum-blonde, long wavy hair|Preserved|
|Green eyes|Preserved|
|Clothing and colours|Preserved|
|Jewellery|Preserved|
|Coffee cup|Preserved|
|Crossed-leg pose|Corrected|
|Café and bar|Both represented|
|Wine bottles|Preserved|
|Photographic medium|Corrected|
|Depth of field and bokeh|Preserved and expressed coherently|
|No explanatory preamble|Satisfied|
|No invented scene content|Not completely satisfied|

### Remaining limitation

The output introduced a café table even though the input did not explicitly specify one.

This was a plausible contextual inference, but technically violated the rule against inventing objects.

The decision was made not to tighten the prompt further at this stage.

The inference did not materially undermine the scene, whereas earlier errors involving pose and composition had substantially changed the requested image.

The v1.3 prompt was therefore frozen as the working baseline.

## 10. Image-Generation Comparison

The final system prompt was subsequently used in a controlled Krea 2 comparison.

The experiment used two prompt sources:

- A detailed, carefully authored natural-language prompt
    
- A sparse Danbooru-style prompt
    

Each source was tested with enhancement enabled and disabled.

An untouched workflow baseline was also included.

### Output files

|Filename|Purpose|
|---|---|
|`workflow-cui-user_00001_.png`|Original detailed user prompt|
|`workflow-enh-user_00001_.png`|Enhanced detailed user prompt|
|`workflow-cui-shit_00001_.png`|Raw Danbooru prompt|
|`workflow-enh-shit_00001_.png`|Enhanced Danbooru prompt|
|`workflow-normal_00001_.png`|Untouched baseline|

### Findings

**Detailed user prompt:** Enhancement produced a different interpretation but did not demonstrate a clear improvement over the original carefully authored prompt.

**Danbooru prompt:** Enhancement produced a more coherent photographic interpretation, with stronger integration of the subject, environment and scene details.

**Untouched baseline:** The existing workflow already produced strong results, confirming that prompt enhancement was not inherently necessary for high-quality Krea 2 output.

The comparison supported the conclusion that enhancement is most useful when the source input is sparse, fragmented or tag-based.

It did not establish that enhancement universally improves image quality.

For the complete experiment, see [[Krea 2 Prompt Enhancement — Comparative Testing]].

## 11. Recommended Operating Pattern

The resulting working approach is:

```
Detailed, carefully authored prompt
              |
              v
        Enhancement OFF
              |
              v
          Krea 2
```

```
Sparse prose / Danbooru tags
              |
              v
        Enhancement ON
              |
              v
          Krea 2
```

These are defaults rather than absolute rules.

Enhancing an already detailed prompt may still be useful when exploring alternative interpretations.

However, it also introduces another opportunity for semantic drift.

### Why the bypass matters

A switchable enhancer provides three practical benefits:

1. Original prompts remain available without modification.
    
2. Enhancement can be evaluated independently.
    
3. The user retains control over whether Qwen is allowed to reinterpret the description.
    

The enhancer should therefore remain optional.

## 12. Known Limitations

### Semantic fidelity is not guaranteed

Even with explicit instructions, Qwen may occasionally reinterpret a pose, object or relationship.

Text outputs should be inspected when exact details matter.

### Enhancement can introduce content

The café-table example demonstrates that plausible additions can occur despite instructions prohibiting them.

### Sampling may introduce variation

The current sampling settings allow variation in generated text.

The stability of alternative temperature settings has not yet been established.

### Longer prompts are not necessarily better

The unsuccessful long-form system prompt demonstrated that adding more instructions can reduce effective compliance.

### Photographic detail remains conservative

The final system prompt allows restrained enrichment, but does not guarantee that Qwen will introduce meaningful additional texture or lighting descriptions.

### Model-specific behaviour

These observations apply to the tested Qwen3-VL 4B configuration.

Other text-generation models, quantisations, inference nodes or parameter settings may behave differently.

## 13. Future Experiments

Potential follow-up work includes:

- Testing lower sampling temperatures for more deterministic translation
    
- Comparing multiple generations from identical input
    
- Testing more complex Danbooru compositions
    
- Evaluating multiple subjects and spatial relationships
    
- Testing fantasy anatomy, armour and weapons
    
- Comparing image-to-Danbooru-to-Krea results across different reference-image styles
    
- Evaluating whether enhanced prompts improve consistency across seeds
    
- Reusing the component with other image-generation architectures
    

A particularly useful future test would involve screenshots of Final Fantasy XIV or World of Warcraft characters.

These introduce complex clothing, non-human anatomical features, fantasy materials and game-specific visual conventions.

The objective would be to determine whether the enhancer can preserve character-defining features while converting game imagery into coherent photographic or illustrative descriptions.

## 14. Key Lessons

1. **Faithfulness is more important than creativity** when translating structured visual tags.
    
2. Explicit examples are useful for preventing ambiguous pose and spatial interpretations.
    
3. A compact system prompt may outperform a more elaborate one.
    
4. Detailed prose often needs little or no enhancement.
    
5. Danbooru tags benefit from conversion into connected natural language.
    
6. Generated prompt text should remain visible and reusable.
    
7. Prompt enhancement should be optional and bypassable.
    
8. Textual correctness and final image quality must be evaluated separately.
    
9. Model sampling behaviour is part of prompt-enhancement reproducibility.
    
10. The working v1.3 system prompt should remain unchanged until a specific new failure justifies revision.
    

## Related Notes

- [[ComfyUI — Workflow Architecture & Design Patterns]]
    
- [[Krea 2 Prompt Enhancement — Comparative Testing]]
    
- [[ComfyUI — Prompt Conditioning — Subgraph Design]]
    
- [[ComfyUI — Hybrid WD14 + Qwen Image-to-Illustrious Prompt Pipeline]]
    

## Conclusion

The Qwen3-VL prompt-enhancement component evolved from a general-purpose prompt expander into a more focused semantic translation layer.

Its principal value is converting fragmented or tag-based visual descriptions into coherent natural-language prompts while attempting to preserve the original scene.

The successful v1.3 prompt provides a practical baseline, but does not eliminate semantic drift or guarantee improved image quality.

The most effective design is therefore a **switchable, inspectable prompt-enhancement stage** that exposes its final text and integrates cleanly with downstream Krea 2 conditioning.

This preserves both automation and user control.