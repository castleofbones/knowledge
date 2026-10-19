---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- image-generation
    
- illustrious
    
- danbooru
    
- wd14
    
- qwen3-vl
    
- custom-nodes
    
- prompt-engineering
    
- workflow-architecture
    

purpose: knowledge
---
## Overview

**Implementation:** ComfyUI Desktop on Windows 11  
**Repository:** `ComfyUI-Prompt-Tools`  
**Status:** Working end-to-end; semantic refinement pending  
**Primary models:** WD EVA02 Tagger v3 and Qwen3-VL 4B  
**Output:** Categorised, formatted Illustrious prompt

This workflow converts an input image into a structured Illustrious prompt by combining two independent image-analysis systems.

WD14 supplies established Danbooru vocabulary, while Qwen3-VL contributes richer visual descriptions, environmental details, lighting and stylistic information.

The outputs are merged, deduplicated, classified into semantic categories and formatted using deterministic custom Python nodes.

The architecture deliberately separates visual interpretation, semantic classification and string processing.

## 1. Architecture

```
                           Input Image
                                |
                  +-------------+-------------+
                  |                           |
                  v                           v
             WD14 EVA02                Image Resize
                  |                           |
                  |                           v
                  |                      Qwen Pass 1
                  |                           |
                  |                           v
                  |                    Danbooru Tag Cleaner
                  |                           |
                  +-------------+-------------+
                                |
                                v
                        Danbooru Tag Merger
                                |
                                v
                        Danbooru Tag Batcher
                           20 tags/batch
                                |
                  +-------------+-------------+
                  |             |             |
                  v             v             v
             Qwen Pass 2A  Qwen Pass 2B  Qwen Pass 2C
                  |             |             |
                  +-------------+-------------+
                                |
                                v
                       Classification Join
                                |
                                v
                     Illustrious Tag Formatter
                                |
                                v
                       Final Prompt + Audit
```

### Custom nodes

The `ComfyUI-Prompt-Tools` repository contains four custom nodes:

|Node|Responsibility|
|---|---|
|Danbooru Tag Cleaner|Extract structured tags from noisy Qwen output|
|Danbooru Tag Merger|Combine WD14 and Qwen tags, removing exact duplicates|
|Danbooru Tag Batcher|Divide merged tags into classification batches|
|Illustrious Tag Formatter|Categorise output groups and construct the final prompt|

Each node has a narrowly defined responsibility.

## 2. Image Analysis

### WD14 — Canonical vocabulary

**Selected model:** `wd-eva02-large-tagger-v3`

Four WD14 v3 models were evaluated using the same Vampy Goth image:

- EVA02 Large
    
- ConvNeXt
    
- SwinV2
    
- ViT
    

EVA02 was selected because it provided the richest useful canonical vocabulary in this comparison.

Its strengths include standard Danbooru tags for subjects, anatomy, clothing, facial features and camera framing.

Examples:

```
1girl, solo, looking_at_viewer, yellow_eyes,
black_sclera, black_lips, lipstick, makeup,
speech_bubble, hair_behind_ear
```

**Limitation:** WD14 can produce incorrect character/franchise identifications and metadata tags.

The selected model is therefore a working choice based on the validation image, not a claim of universal superiority.

### Qwen3-VL — Visual enrichment

**Model:** `qwen3vl_4b_fp8_scaled.safetensors`

Qwen analyses the image independently and produces descriptive Danbooru-style tags.

It contributes details that WD14 frequently misses:

```
lace_collar, high_neckline, illuminated_windows,
urban_night_background, glossy_hair,
dramatic_shading, soft_lighting, chiaroscuro
```

#### Pass 1 settings

|Parameter|Value|
|---|---|
|max_length|512|
|Sampling Mode|ON|
|Temperature|0.7|
|Top K|64|
|Top P|0.95|
|Min P|0.05|
|Repetition Penalty|1.15|
|Presence Penalty|0|
|Seed|0|
|Thinking|OFF|
|use_default_template|ON|

#### Pass 1 prompt

```
Analyse the attached image and identify useful Danbooru-style tags that describe its visible content.

Describe concrete visible features that would help recreate a similar image.

Include relevant tags for:
- subjects and subject count
- hair
- eyes and anatomical features
- facial expression
- body characteristics
- clothing and accessories
- pose and camera framing
- environment and background
- art/rendering style
- lighting

Use lowercase Danbooru-style tags with underscores.

Prefer specific visible details over abstract concepts or interpretations.
Do not describe things that are absent.
Do not invent artist names, character names, series names or other identities unless clearly recognizable.
Avoid redundant variations of the same concept.
Aim for approximately 30-50 useful tags.

Output ONLY one comma-separated line of tags.
No headings, notes, explanations or other text.
```

### Image conditioning

The Qwen branch uses the native `Scale Image to Max Dimension` node.

|Setting|Value|
|---|---|
|upscale_method|`area`|
|largest_size|`1936`|

This was established experimentally after particular image dimensions caused Qwen to return blank output.

Tests showed that 1920 could fail while 1919, 1921, 1928 and 1936 succeeded on a problematic image.

The exact underlying preprocessing issue remains unconfirmed.

**1936 is the currently validated working configuration.**

## 3. Why WD14 and Qwen Run Independently

An experiment supplied WD14's candidate tags to Qwen as vocabulary guidance.

The intended behaviour was for Qwen to retain accurate canonical tags while adding missing details.

Instead, Qwen largely repeated WD14's output and contributed little additional information.

It also occasionally focused on text visible within the image rather than performing the requested tagging task.

This demonstrated an undesirable anchoring effect.

### Architectural decision

Keep WD14 and Qwen independent.

Merge their outputs only after both have completed image analysis.

This preserves the complementary strengths of both systems without allowing either to constrain the other's observations.

## 4. Danbooru Tag Cleaner

The cleaner extracts a reliable comma-separated tag list from Qwen output.

It handles:

- Leading `assistant` markers
    
- Markdown separators
    
- `Tags:` headings
    
- Explanatory prose surrounding tag lists
    
- Quoted text associated with known text-bearing tags
    

Example input:

```
**Note:** The image contains a speech bubble.

---

white_hair, yellow_eyes, speech_bubble, "you_want_me?"
```

Output:

```
white_hair, yellow_eyes, speech_bubble("you_want_me?")
```

The cleaner deliberately avoids semantic correction or deduplication.

When no reliable tag list can be identified, it returns an empty string and an audit message.

## 5. Danbooru Tag Merger

The merger combines the two independent outputs.

### Merge policy

1. WD14 tags appear first.
    
2. Qwen tags are appended.
    
3. Exact duplicates are removed case-insensitively.
    
4. The first occurrence is retained.
    
5. Input order is preserved.
    
6. No fuzzy matching or semantic deduplication is performed.
    

Example:

```
WD14:
white_hair, yellow_eyes, pointy_ears

Qwen:
white_hair, long_white_hair, pointed_ears
```

Merged:

```
white_hair, yellow_eyes, pointy_ears,
long_white_hair, pointed_ears
```

`pointy_ears` and `pointed_ears` remain separate because they are not exact duplicates.

Likewise, `white_hair` and `long_white_hair` remain separate because the second tag contains additional information.

This behaviour is intentional.

## 6. Semantic Classification

Qwen Pass 2 receives the merged tags as text and assigns each tag exactly one semantic category.

It does not receive the original image.

### Categories

|Category|Purpose|
|---|---|
|SUBJECT|Subject count, type and species|
|COPYRIGHT|Named franchise or series|
|CHARACTER|Specifically named fictional character|
|FEATURES|Facial, anatomical and fantasy features|
|HAIR|Hair colour, length and hairstyle|
|EXPRESSION_AGE|Expression, emotion and apparent age|
|BODY|Build, physique and proportions|
|OUTFIT|Clothing, footwear and accessories|
|FRAMING_POSE|Framing, gaze, viewing angle and pose|
|ENVIRONMENT|Scenery, locations and background objects|
|STYLE_LIGHTING|Rendering style, medium and lighting|
|UNKNOWN|Useful tags without a confident category|
|REMOVE|Metadata, absence tags and nonvisual concepts|

### Pass 2 settings

|Parameter|Value|
|---|---|
|max_length|1024|
|Sampling Mode|OFF|
|Thinking|OFF|
|use_default_template|ON|

### Complete Pass 2 prompt

```
Classify each comma-separated Danbooru image tag below.

Assign every tag exactly ONE category.
Preserve each tag exactly.
Do not invent, merge, duplicate or omit tags.
Use UNKNOWN when uncertain.
Ignore any text such as "assistant", "Tags:", headings or notes that is not an image tag.

Categories:

SUBJECT = generic subject type/count: girl, 1girl, boy, solo, 2girls, female_character
COPYRIGHT = named series, franchise, game, anime or manga
CHARACTER = specifically named fictional character, not generic subject descriptions
FEATURES = eyes, facial/anatomical and fantasy/non-human features
HAIR = hair colour, length, texture or hairstyle
EXPRESSION_AGE = facial expression, facial emotion or apparent age
BODY = build, physique, proportions or physical conditioning
OUTFIT = clothing, footwear, jewellery or worn accessories
FRAMING_POSE = framing, viewing angle, gaze, orientation or physical pose
ENVIRONMENT = location, scenery, background or environmental objects
STYLE_LIGHTING = medium, rendering style, visual effects or lighting
UNKNOWN = useful descriptive tag that does not clearly fit elsewhere
REMOVE = quality/rating metadata, absence tags or abstract non-visual concepts

Examples:
masterpiece | REMOVE
rating_safe | REMOVE
1girl | SUBJECT
red_hair | HAIR
green_eyes | FEATURES
smiling | EXPRESSION_AGE
toned_body | BODY
pink_leggings | OUTFIT
looking_at_viewer | FRAMING_POSE
gym | ENVIRONMENT
anime_style | STYLE_LIGHTING
pokemon | COPYRIGHT
squirtle | CHARACTER
no_watermark | REMOVE

Output exactly one line per input tag:

tag | CATEGORY

Output ONLY classified tag lines. No headings, notes or explanations.

INPUT TAGS:
```

The batch contents are appended after `INPUT TAGS:`.

## 7. Classification Batching

The hybrid architecture produces approximately 50–60 tags for the current validation images.

The original single-classifier implementation sometimes returned blank output for larger inputs.

Increasing `max_length` from 1024 to 2048 did not resolve the initial Bloke failure.

### Current implementation

`Danbooru Tag Batcher`

|Setting|Value|
|---|---|
|batch_size|20|
|Number of outputs|3|
|Maximum capacity|60 tags|
|Overflow handling|Audit warning|

For 57 merged tags:

```
Input: 57 tags
Batch size: 20
Batch 1: 20 tags
Batch 2: 20 tags
Batch 3: 17 tags
No overflow.
```

Each batch is classified by an independent `TextGenerate` node.

The resulting classification strings are joined using:

```
<<<BATCH>>>
```

The formatter converts this delimiter into line breaks before parsing.

### Important implementation detail

Changing a custom node's Python default does not overwrite widget values already saved in a ComfyUI workflow.

During testing, the batcher continued using `batch_size = 30` despite the Python default having changed to 20.

The saved widget value had to be changed explicitly.

### Diagnostic findings

Several blank-generation failures were investigated.

Tests established that:

- Individual tag subsets could classify successfully.
    
- Increasing output length did not resolve the original failure.
    
- A malformed prompt connection could cause Qwen to generate descriptive prose instead of classifications.
    
- An existing `TextGenerate` instance returned blank output even after other generation nodes were disabled.
    
- Recreating the problematic node resolved the final failure.
    

**Conclusion:** The three-way 20-tag implementation is working, but the evidence does not conclusively establish a universal Qwen input-size limit or a single root cause for every blank-generation incident.

## 8. Illustrious Tag Formatter

The formatter performs deterministic final prompt construction.

### Fixed header

```
masterpiece, best quality, very aesthetic, absurdres
```

The header is inserted exactly once.

### Formatting rules

- Group tags by semantic category.
    
- Preserve the established category order.
    
- Replace underscores with spaces.
    
- Separate tags within each group using `,` .
    
- Separate nonempty groups using `,\n\n`.
    
- Omit empty categories.
    
- Exclude REMOVE tags.
    
- Include UNKNOWN tags in the output and audit.
    
- Report malformed classification lines separately.
    

No rating tag is added automatically.

### Example output

```
masterpiece, best quality, very aesthetic, absurdres,

1girl, solo,

purple eyes, pointy ears, pointed ears,

long hair, white hair, long white hair,

leather jacket, lace top, choker,

looking at viewer, close up,

city street, neon signs, night sky,

digital art, volumetric lighting
```

The example deliberately preserves semantic overlaps that have not yet been normalised.

## 9. Validation Results

|Image|WD14|Qwen|Unique merged|Duplicates|
|---|---|---|---|---|
|Vampy Goth|33|29|57|5|
|Original Elf|33|25|55|3|
|Ornate Elf|31|20|47|4|
|Waterfall|38|26|60|4|
|Bloke|38|23|57|4|
|Gym|35|24|54|5|

### Vampy Goth

WD14 contributed canonical tags such as `black_sclera`, `black_lips`, `makeup` and `lipstick`.

Qwen added `lace_collar`, `high_neckline`, `illuminated_windows`, `dramatic_shading` and `chiaroscuro`.

This provided the clearest initial evidence that the independent branches were complementary.

### Original Elf

WD14 contributed subject, anatomical and clothing vocabulary.

Qwen contributed neon-city details, environmental descriptions and rendering information.

### Ornate Elf

WD14 produced `triforce` and `princess_zelda` for an image being used as a generic fantasy elf portrait.

This demonstrated that named character and copyright predictions require stronger validation than ordinary visual tags.

### Waterfall

WD14 recognised Pokémon-related concepts.

Qwen contributed additional descriptions of vegetation, waterfalls, pose, clothing and lighting.

### Bloke

The initial full classification returned only the fixed Illustrious header because Pass 2 produced no usable classification output.

After implementing batching and resolving node issues, Bloke successfully passed through the entire pipeline.

### Gym

The hybrid workflow produced useful anatomical, clothing, gym-equipment and rendering tags.

It also exposed redundancy and unwanted metadata.

## 10. Outstanding Quality Issues

### Canonical aliases

Examples:

```
pointy_ears / pointed_ears
close-up / close_up
spiked_hair / spiky_hair
earrings / ear_rings
```

These are candidates for deterministic normalisation.

### General and specific descriptions

Examples:

```
white_hair / long_white_hair
jacket / black_jacket / leather_jacket
smile / gentle_smile
freckles / body_freckles
```

These are not necessarily duplicates.

Normalisation must distinguish redundant aliases from tags that add useful specificity.

### Weak or invented vocabulary

Examples:

```
sharp_sharpness
flame_flames
hair_winds
facial_expression
```

These require investigation before defining correction rules.

### Incorrect identity predictions

Examples:

```
princess_zelda
triforce
lana_(pokemon)
```

Named identities should not automatically be accepted solely because WD14 predicted them.

### Metadata leakage

Examples:

```
artist_name
patreon_username
high_resolution
```

These should generally be classified as REMOVE when they do not describe useful visible content.

### Classification errors

Examples include:

- `pyrokinesis` classified as FEATURES.
    
- `english_text` classified UNKNOWN.
    
- Useful Bloke tags missing from the final prompt.
    
- Escaped Danbooru syntax altered during underscore replacement.
    

These issues should be diagnosed from the complete classifier outputs and formatter audit before introducing additional processing.

## 11. Next Steps

- Inspect the complete three-batch Bloke classification output.
    
- Identify why some classified tags are absent from the final prompt.
    
- Define canonical alias normalisation.
    
- Distinguish redundant aliases from general/specific descriptions.
    
- Improve treatment of identity and copyright predictions.
    
- Remove unwanted metadata consistently.
    
- Preserve escaped Danbooru syntax correctly.
    
- Improve validation of Qwen-generated pseudo-tags.
    
- Extend audit reporting for missing and malformed classifications.
    
- Validate additional images without changing the established architecture.
    
- Consider dynamic batching only if current capacity becomes a practical limitation.
    
- Commit and preserve the working workflow and custom-node implementation in Git.
    

## 12. Architectural Decisions

### Keep WD14 and Qwen independent

**Reason:** Feeding WD tags into Qwen caused anchoring and reduced enrichment.

### Keep the merger simple

**Reason:** Exact duplicates can be removed deterministically; semantic equivalence requires a separate policy.

### Separate extraction from classification

**Reason:** Combining image understanding, classification and formatting in one Qwen prompt reduced reliability.

### Use Python for deterministic processing

**Reason:** Custom nodes provide clearer behaviour and audits than complicated ComfyUI regex/list/loop chains.

### Retain the working three-way batching implementation

**Reason:** It has successfully processed the previously failing Bloke case.

### Preserve diagnostic outputs

**Reason:** Cleaner, merger, batcher and formatter audits make it possible to locate failures at specific processing stages.

## 13. Session Outcome

The hybrid WD14 + Qwen image-to-Illustrious workflow has been demonstrated working end-to-end.

The major engineering problems of independent extraction, structural cleaning, deterministic merging, batching and final formatting have working implementations.

Remaining work is primarily concerned with semantic quality, normalisation, classification completeness and validation.

**Key lesson:** WD14 provides established vocabulary; Qwen provides richer visual understanding. Keeping them independent produces more useful combined output than forcing either model to perform both jobs.

**Secondary lesson:** Before redesigning a workflow to accommodate mysterious model behaviour, verify the actual node instance, saved widget values and connections. Sometimes deleting and recreating the offending ComfyUI node really does solve the problem.