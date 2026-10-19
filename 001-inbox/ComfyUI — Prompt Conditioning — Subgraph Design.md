---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- krea-2
    
- workflow-architecture
    
- subgraphs
    
- prompt-conditioning
    
- qwen3-vl
    
- lora
    

purpose: knowledge
---
## Overview

The Prompt Conditioning component is intended to encapsulate prompt-source selection, optional Qwen3-VL enhancement and text conditioning within a reusable ComfyUI subgraph.

It originated from a working experimental implementation on the main workflow canvas, developed to compare natural-language prompts against Danbooru-style tags, both with and without enhancement.

The component's primary responsibility is to produce **the exact positive-prompt text selected for generation and its corresponding CONDITIONING output**.

It should not be responsible for image generation, VAE decoding, eye detailing, inpainting or output saving.

> [!important] Design principle  
> Prompt preparation and image generation are separate responsibilities. The conditioning component should not need to know how its resulting conditioning will be used.

## 1. Objectives

The component is intended to:

1. Accept alternative textual prompt sources.
    
2. Select the desired source.
    
3. Optionally enhance the selected prompt using Qwen3-VL.
    
4. Preserve the original prompt as a bypass path.
    
5. Encode the final selected text using the appropriate CLIP context.
    
6. Expose the final prompt text independently of conditioning.
    
7. Produce positive conditioning for downstream image-generation nodes.
    
8. Support integration with the existing multi-LoRA architecture.
    
9. Remain independently testable before encapsulation.
    
10. Support future image-to-prompt-to-image workflows.
    

### Out of scope

The component does not perform:

- Image generation or sampling
    
- VAE encoding or decoding
    
- Eye detailing
    
- Manual inpainting
    
- Image upscaling
    
- Image saving
    
- Danbooru tag extraction from source images
    

Image-to-Danbooru processing is a separate upstream responsibility.

## 2. Implementation Status

The design combines experimentally validated behaviour with planned architectural changes.

|Capability|Status|
|---|---|
|Natural-language prompt input|Tested|
|Danbooru prompt input|Tested|
|Prompt-source switching|Tested|
|Qwen3-VL enhancement|Tested|
|Enhancement bypass|Tested|
|Shared downstream CLIP encoding|Tested|
|Fixed-seed comparative generation|Tested|
|Final prompt text output|Planned subgraph interface|
|Conditioning output|Planned subgraph interface|
|Multi-LoRA integration|Planned|
|Subgraph encapsulation|Planned|
|Reuse across multiple workflows|Not yet validated|

The experimental processing paths are functional.

The final subgraph interface and complete encapsulation have not yet been established as implemented.

The intended architecture includes three switching decisions or controls, but only the roles of the prompt-source and enhancement switches are established here.

The third switch should be documented from the actual workflow before its purpose is treated as confirmed.

## 3. Component Context

The intended high-level architecture is:

```
                  ┌───────────────────────┐
                  │ Image → Danbooru      │
                  │ Separate component    │
                  └───────────┬───────────┘
                              │
                              │ TEXT
                              ▼
┌──────────────────────────────────────────────────┐
│                PROMPT CONDITIONING               │
│                                                  │
│  Prompt Selection                                │
│          │                                       │
│          ▼                                       │
│  Optional Qwen Enhancement                       │
│          │                                       │
│          ▼                                       │
│  Final Prompt Text ────────────────┐             │
│          │                         │             │
│          ▼                         │             │
│  CLIP Text Encode                  │             │
│          │                         │             │
└──────────┼─────────────────────────┼─────────────┘
           │                         │
           ▼                         ▼
     CONDITIONING             POSITIVE_PROMPT
           │                         │
           ▼                         ▼
    Base Generation           Inspect / Copy /
                              Refine / Reuse
```

This establishes a clear boundary between semantic extraction, prompt preparation and image synthesis.

## 4. Inputs

### Natural-language prompt

A user-authored description intended for direct image generation or optional enhancement.

The input may already contain detailed information about:

- Subjects and appearance
    
- Clothing and materials
    
- Pose and expression
    
- Objects and environment
    
- Lighting
    
- Composition
    
- Photographic or illustrative style
    

Detailed prompts should be preserved when enhancement is disabled.

### Danbooru prompt

A comma-separated list of descriptive tags.

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

The component does not need to know whether the tags were written manually or extracted from an image.

This allows the planned [[Image-to-Danbooru Pipeline — Architecture & Processing]] to operate independently.

### Qwen model connection

The enhancement operation requires the applicable Qwen text-generation model connection.

The existing Krea 2 template provides this capability.

The intention is to reuse the model connection rather than introduce a redundant model load.

### CLIP connection

The conditioning operation requires the appropriate CLIP/text-encoder object.

The CLIP connection may be modified by compatible LoRAs before encoding.

### Model connection

Where LoRA integration is included within the component boundary, the corresponding MODEL connection must also be considered.

A LoRA loader may return modified MODEL and CLIP objects.

Both outputs must be routed to the relevant downstream consumers.

### Control inputs

At minimum, the interface requires controls for:

- Prompt-source selection
    
- Enhancement enabled or disabled
    

The final interface may expose additional controls where justified by the implementation.

The aim is to avoid exposing every internal parameter unnecessarily.

## 5. Outputs

### POSITIVE_PROMPT

**Type:** TEXT / STRING

Contains the final selected positive-prompt text.

When enhancement is disabled, this should be the selected original prompt.

When enhancement is enabled, this should be the Qwen-generated description.

This output must represent the text actually passed into CLIP Text Encode.

#### Uses

- Previewing the final prompt
    
- Copying it into another workflow
    
- Manual refinement
    
- Saving or logging prompts
    
- Comparing original and enhanced text
    
- Reusing text in subsequent generation stages
    

The output is intentionally independent of image generation.

### CONDITIONING

**Type:** CONDITIONING

Contains the conditioning produced by CLIP Text Encode using the selected positive-prompt text and applicable CLIP context.

This output feeds the image-generation path.

### Potential MODEL output

If the component includes the multi-LoRA loader internally, it may also need to expose the modified MODEL object.

This is not merely an optional convenience.

The sampler must receive the model with the intended LoRAs applied.

The final component boundary should therefore be chosen carefully.

Two valid architectural approaches are:

**Approach A — LoRA loading outside Prompt Conditioning**

The upstream loader supplies the modified MODEL to the sampler and modified CLIP to Prompt Conditioning.

**Approach B — LoRA loading inside Prompt Conditioning**

The component exposes the modified MODEL alongside the positive prompt and conditioning.

The preferred approach remains an implementation decision.

## 6. Prompt Source Selection

The first established switching decision chooses between two textual inputs.

```
Natural-Language Prompt ───┐
                           ├── Source Switch
Danbooru Tags ─────────────┘
                                  │
                                  ▼
                           Selected Text
```

This provides a common downstream interface regardless of the prompt's origin.

### Design rationale

The conditioning component should not need separate encoding pipelines for different prompt formats.

Both inputs ultimately become text.

Selection should therefore occur before enhancement and encoding.

This simplifies downstream wiring and supports controlled comparison.

## 7. Optional Qwen Enhancement

The selected prompt is routed through two alternative paths.

```
Selected Text ───────────────┐
                             │
                             ▼
                      Qwen3-VL Enhance
                             │
                             ▼
                      Enhanced Text
                             │
Selected Text ───────────────┤
                             │
                             ▼
                      Enhancement Switch
                             │
                             ▼
                       Final Prompt
```

The switch selects either the original text or Qwen's rewritten text.

### Enhancement disabled

The selected prompt is passed directly to CLIP Text Encode.

No semantic rewriting is requested.

### Enhancement enabled

The selected prompt is processed using Qwen3-VL and the custom v1.3 system prompt.

Qwen converts the description into a cohesive natural-language image prompt.

The enhancer is intended to preserve:

- Subject identity and appearance
    
- Clothing and colours
    
- Pose and expression
    
- Objects and scene content
    
- Spatial relationships
    
- Lighting and composition
    
- Requested visual medium
    

The enhancement path is not intended to invent arbitrary details.

### Established behaviour

Testing showed that enhancement was most useful for sparse Danbooru-style prompts.

Detailed natural-language prompts frequently performed equally well without enhancement.

Consequently, enhancement remains optional.

See [[Qwen3-VL Prompt Enhancement for Krea 2]] and [[Krea 2 Prompt Enhancement — Comparative Testing]].

## 8. CLIP Text Encoding

After enhancement selection, the final prompt is passed into CLIP Text Encode.

```
Final Prompt TEXT ───────┐
                         │
                         ▼
                    CLIP Text Encode
                         ▲
                         │
                    Modified CLIP
                         │
                         ▼
                    CONDITIONING
```

The text input and CLIP object represent separate data paths.

The prompt is not itself modified by LoRA loading.

Instead, LoRA application may modify the model and/or text encoder used to interpret the prompt.

### Important distinction

The Qwen system prompt controls text generation.

The final positive prompt controls image conditioning.

They are not interchangeable.

The Qwen system prompt must not accidentally become part of the Krea 2 positive-prompt text.

## 9. Multi-LoRA Integration

The existing workflow architecture includes LoRA loading, with plans to integrate a multi-LoRA loader into the conditioning design.

Conceptually:

```
MODEL ─────┐
           │
CLIP ──────┼── Multi-LoRA Loader
           │          │
LoRA Stack ┘          ├── Modified MODEL ──► Sampler
                      │
                      └── Modified CLIP
                                 │
                                 ▼
                          CLIP Text Encode
                                 ▲
                                 │
                          Final Prompt TEXT
                                 │
                                 ▼
                            CONDITIONING
                                 │
                                 ▼
                              Sampler
```

This illustrates the relationship between the model path, CLIP path and selected prompt.

### Design considerations

- The LoRA loader must be compatible with the chosen model architecture.
    
- A loader's MODEL and CLIP outputs must be connected to the correct consumers.
    
- LoRA strength and stacking behaviour depend on the implementation.
    
- Multiple LoRAs can interact; their effects should not be assumed to be independent or simply additive.
    
- The prompt-enhancement stage should not be confused with LoRA application.
    

### Placement

The phrase _before CLIP Text Encode_ refers to the CLIP processing path.

It does not imply that the LoRA loader receives the enhanced prompt as its input.

The text and model/CLIP paths remain distinct.

The final decision about whether LoRA loading belongs inside the subgraph should be made when the component interface is finalised.

## 10. Proposed Internal Architecture

The intended processing arrangement is:

```
                  NATURAL-LANGUAGE PROMPT
                            │
                            │
                            ▼
                       ┌────────┐
DANBOORU TAGS ────────►│ SOURCE │
                       │ SWITCH │
                       └───┬────┘
                           │
                           ▼
                     SELECTED TEXT
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 │                   ▼
                 │             QWEN3-VL
                 │             ENHANCER
                 │                   │
                 │                   ▼
                 │             ENHANCED TEXT
                 │                   │
                 ▼                   ▼
                 ┌───────────────────┐
                 │ ENHANCEMENT       │
                 │ SWITCH            │
                 └─────────┬─────────┘
                           │
                           ▼
                    FINAL PROMPT TEXT
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
         POSITIVE_PROMPT        CLIP TEXT ENCODE
                                      ▲
                                      │
                                 MODIFIED CLIP
                                      │
                                      ▼
                                 CONDITIONING
```

The diagram intentionally focuses on the established text-processing path.

Model loading and LoRA integration remain separate architectural concerns until their final placement is decided.

## 11. Relationship to Image Generation

The component should produce conditioning without assuming a particular downstream image-processing branch.

The established workflow includes independent image outputs:

```
Base Generation ──────┐
                      │
Eye Detailer ─────────┼── Switch (Any) ──► Save Image
                      │
Manual Inpaint ───────┘
```

These branches are alternatives feeding the final output switch.

They are not a mandatory sequential chain.

The Prompt Conditioning component should remain upstream of the generation operation and should not control which completed IMAGE branch is selected for saving.

This preserves separation between prompt preparation and image-output routing.

## 12. Reusability

The intended subgraph should support several usage patterns.

### Hand-written prompt

```
User Prompt
     │
     ▼
Prompt Conditioning
     │
     ▼
Image Generation
```

### Danbooru prompt

```
Danbooru Tags
     │
     ▼
Prompt Conditioning
     │
     ▼
Image Generation
```

### Image-derived prompt

```
Reference Image
     │
     ▼
Image → Danbooru
     │
     ▼
Prompt Conditioning
     │
     ▼
Image Generation
```

### Iterative reinterpretation

```
Reference Image
     │
     ▼
Image → Danbooru
     │
     ▼
Prompt Conditioning
     │
     ▼
Initial Generation
     │
     ▼
Image Analysis / Prompt Extraction
     │
     ▼
Manual Refinement
     │
     ▼
Prompt Conditioning
     │
     ▼
Final Generation
```

These patterns motivate exposing the final prompt as text.

Without that output, the component would be useful primarily as a conditioning operation rather than as a reusable prompt-processing stage.

## 13. Testing Strategy

The existing experimental implementation should remain the reference until encapsulation is complete.

### Baseline tests

**Test 1 — Detailed prose, enhancement disabled**

Verify that the final text matches the selected user prompt and produces the expected conditioning.

**Test 2 — Detailed prose, enhancement enabled**

Verify that the Qwen output is selected and passed to CLIP encoding.

**Test 3 — Danbooru tags, enhancement disabled**

Verify that the original tag list reaches CLIP encoding without rewriting.

**Test 4 — Danbooru tags, enhancement enabled**

Verify that the Qwen-generated prose reaches CLIP encoding.

### Output consistency

For each test:

- Inspect the final positive-prompt text.
    
- Confirm that the text passed to CLIP is the same as the exposed output.
    
- Confirm that the correct conditioning reaches the generation path.
    
- Verify switch routing.
    
- Preserve the fixed seed and generation settings when comparing images.
    

### LoRA tests

After integrating the multi-LoRA loader:

- Verify the modified CLIP connection.
    
- Verify the modified MODEL connection.
    
- Confirm that disabling LoRAs restores the expected baseline.
    
- Test one LoRA before testing multiple LoRAs.
    
- Preserve all unrelated settings during comparisons.
    

### Encapsulation test

Once the processing path is stable, encapsulate it as a subgraph.

Compare the output of the encapsulated version with the working main-canvas implementation.

The objective is to confirm that encapsulation has not changed routing, prompt selection or conditioning behaviour.

## 14. Known Risks and Constraints

### Semantic drift

Qwen may reinterpret scene relationships, poses or framing.

Enhancement must remain bypassable.

### Hidden intermediate behaviour

Premature encapsulation can make debugging harder.

Useful previews should be retained while the subgraph is being developed.

### LoRA model consistency

If LoRA loading is incorporated into the component, the modified MODEL must reach the sampler.

Exposing only conditioning could leave the generation path using an unmodified model.

### Subgraph reuse limitations

A subgraph embedded in one workflow should not be assumed to remain synchronised with copies in other workflows.

The design provides encapsulation, but not necessarily centrally managed versioning.

### Custom-node compatibility

Switches and utilities supplied by third-party extensions may behave differently across versions.

The existing workflow has previously encountered compatibility issues involving Impact Pack switches following a ComfyUI update.

These should be treated as extension/version-specific implementation concerns, not as evidence that the component architecture itself is invalid.

## 15. Outstanding Design Decisions

Before final encapsulation, resolve the following:

- Confirm the role of the third planned switch.
    
- Finalise the subgraph's external input names.
    
- Decide whether multi-LoRA loading belongs inside or outside the component.
    
- Determine whether a modified MODEL output is required.
    
- Confirm how the existing Qwen model connection will be supplied.
    
- Decide which internal preview nodes should remain available.
    
- Establish whether prompt text should be logged or saved independently.
    
- Validate behaviour after subgraph encapsulation.
    

These are implementation decisions, not unresolved problems with the tested prompt-enhancement logic.

## 16. Design Principles

1. **Preserve a single selected prompt.** The text exposed to the user must be the text used for conditioning.
    
2. **Keep enhancement optional.** Qwen should not be a mandatory intermediary.
    
3. **Separate text processing from model modification.** Prompt enhancement and LoRA loading perform different functions.
    
4. **Maintain model/CLIP consistency.** Both sides of a LoRA-modified generation path must be wired correctly.
    
5. **Expose reusable outputs.** Prompt text is a first-class output, not merely an internal intermediate value.
    
6. **Encapsulate after validation.** Preserve the working experimental implementation until the subgraph is verified.
    
7. **Avoid unnecessary coupling.** Image tagging, prompt conditioning and image generation should remain distinct responsibilities.
    
8. **Retain inspectability.** The architecture should support debugging without reconstructing hidden processing paths.
    
9. **Preserve reproducibility.** Fixed seeds and known-good settings should remain available for comparisons.
    
10. **Document implementation separately from intention.** Planned functionality must not be mistaken for tested behaviour.
    

## Related Notes

- [[ComfyUI — Workflow Architecture & Design Patterns]]
    
- [[Qwen3-VL Prompt Enhancement for Krea 2]]
    
- [[Krea 2 Prompt Enhancement — Comparative Testing]]
    
- [[ComfyUI — Hybrid WD14 + Qwen Image-to-Illustrious Prompt Pipeline]]
    

## Conclusion

The Prompt Conditioning subgraph is intended to turn an experimentally validated collection of prompt-processing nodes into a coherent, reusable component.

Its defining feature is the separation of **prompt selection, optional semantic enhancement and text encoding**, while preserving the final selected prompt as an independent output.

The experimental implementation has already demonstrated the value of this separation.

The remaining work concerns component boundaries, multi-LoRA integration, the final switching arrangement and subgraph encapsulation.

The objective is a component that can accept hand-written prompts, Danbooru tags or image-derived descriptions without requiring changes to the downstream image-generation architecture.