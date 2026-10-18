---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - comfyui
  - image-generation
  - krea-2
  - prompt-engineering
  - danbooru
  - natural-language
  - illustrious
  - character-design
  - workflow
---
> [!abstract] Overview  
> Effective prompting is not simply about finding more descriptive words. It is about communicating a visual specification in a form the image-generation model can interpret.
> 
> Our Krea 2 experiments demonstrated the value of combining two complementary approaches:
> 
> - **Danbooru-style tags** for rapidly identifying subjects, attributes, clothing, objects and visual concepts.
>     
> - **Natural language** for describing relationships, spatial arrangements, material construction, lighting, expression and artistic intent.
>     
> 
> Rather than treating these as competing prompting styles, we developed a workflow in which Danbooru tags provide the initial specification and natural language expresses the details that require greater precision.
> 
> The guiding principle is **use concise language for simple attributes and precise language for complex relationships**.

## 1. The Two Prompting Approaches

Image-generation prompts can be written in several different styles.

Two particularly useful approaches are tag-based prompting and natural-language prompting.

### Tag-based prompting

Danbooru-style prompts represent image content as a sequence of descriptive tags.

For example:

`1girl, solo, blonde hair, green eyes, long hair, sweater, pleated skirt, pantyhose, knee boots, sitting, crossed legs, coffee cup`

This provides a compact description of the intended content.

The tags identify:

- Subject
    
- Physical attributes
    
- Clothing
    
- Accessories
    
- Pose
    
- Objects
    

The model must infer many of the relationships between these elements.

### Natural-language prompting

Natural language expresses the intended scene through sentences or descriptive phrases.

For example:

_A blonde woman with long, softly waved hair and green eyes sits with her legs crossed in a quiet café. She wears a fitted sweater, a pleated skirt and knee-high boots, holding a white ceramic coffee cup._

This communicates much of the same content but adds relationships between the elements.

The woman is sitting in the café.

Her legs are crossed.

She is holding the cup.

The clothing belongs to her rather than merely appearing somewhere in the image.

### Comparison

|Tag-based prompting|Natural-language prompting|
|---|---|
|Compact|More expressive|
|Fast to construct|Requires more wording|
|Excellent for attribute inventories|Excellent for relationships|
|Familiar within Danbooru-trained ecosystems|Useful for models responsive to prose|
|Relies heavily on model interpretation|Allows more explicit instructions|
|Convenient for initial character design|Convenient for detailed scene construction|

Neither approach is inherently superior for every model.

The useful distinction is what each representation communicates efficiently.

## 2. Why Start With Danbooru?

Danbooru tags provide a structured vocabulary for visual concepts.

For someone familiar with tag-based prompting, they offer a fast way to establish the intended image without writing elaborate prose.

Consider:

`blonde hair, green eyes, sweater, pleated skirt, boots, coffee cup`

These tags are immediately understandable as an inventory of visual requirements.

### Advantages

**Speed**

A character concept can be assembled rapidly.

**Consistency**

The same vocabulary can be reused across prompts.

**Discoverability**

Tags provide convenient names for poses, clothing, expressions and visual attributes.

**Low linguistic overhead**

The prompt author does not need to construct complete sentences for every detail.

### The limitation

Tags primarily identify concepts.

They do not always explain how those concepts relate to one another.

For example:

`belt, skirt, belt loops`

does not necessarily communicate the intended construction of the clothing.

The model may generate a belt floating over the skirt, a decorative waistband, or a belt attached in an implausible way.

The tags establish the objects, but the physical relationship remains underspecified.

> [!important] Key principle  
> Danbooru tags are an excellent **visual inventory**.
> 
> They are not always a complete specification of the relationships between the items in that inventory.

## 3. Natural Language as a Visual Specification

Natural language becomes particularly useful when the desired result depends on relationships.

These include:

- Spatial relationships
    
- Object construction
    
- Material behaviour
    
- Physical interactions
    
- Lighting relationships
    
- Facial expression
    
- Composition
    
- Narrative intent
    

### Example: A belt

Tag-based description:

`belt, leather belt, pleated skirt, belt loops`

More explicit natural-language description:

_The pleated skirt has sturdy fabric belt loops sewn into its waistband. A narrow brown leather belt passes visibly through the loops, with its metal buckle fastened at the front._

The second description communicates more than the presence of a belt.

It specifies:

1. Where the belt loops are attached.
    
2. What material the belt is made from.
    
3. How the belt passes through the loops.
    
4. Where the buckle is positioned.
    
5. How the components form a coherent garment.
    

These details are important because the model must construct a plausible visual relationship.

### Why this worked for us

During our café portrait experiments, clothing construction became one of the areas where carefully chosen natural-language descriptions provided more useful control than additional isolated tags.

The lesson was not that longer prompts always work better.

It was that **additional words are valuable when they resolve ambiguity**.

## 4. Attribute Precision vs Relationship Precision

Not every visual feature requires the same amount of description.

### Attribute precision

An attribute identifies a property of an object.

Examples:

- Long blonde hair
    
- Green eyes
    
- Black leather boots
    
- White ceramic cup
    
- Charcoal sweater
    

These are relatively simple concepts.

They can often be communicated efficiently through short phrases.

### Relationship precision

A relationship describes how objects, features or materials interact.

Examples:

- A belt threaded through sewn fabric loops
    
- Fingers wrapped around a coffee cup handle
    
- A coat hanging open over a fitted shirt
    
- Hair falling across one shoulder
    
- A boot partially concealed beneath a skirt hem
    
- A pathway leading through a courtyard toward a mansion
    

These often benefit from more explicit language.

### Practical rule

Use short descriptions for straightforward attributes.

Use longer descriptions when the geometry, interaction or construction matters.

This produces prompts that are precise without becoming unnecessarily verbose.

## 5. Avoiding Decorative Prose

Natural-language prompting does not require elaborate creative writing.

A prompt is an instruction to an image-generation model, not a literary performance.

Compare:

_Her lustrous platinum tresses cascade like moonlit silk over her delicate shoulders._

With:

_Long platinum-blonde hair falls in loose waves over her shoulders._

The second description communicates the relevant visual information more directly.

Decorative language may influence mood or style, but it can also introduce ambiguity.

### When poetic language may be useful

Poetic or evocative language can be appropriate when the objective is atmospheric.

Examples include:

- Dreamlike atmosphere
    
- Oppressive gothic architecture
    
- Ethereal moonlight
    
- Melancholic expression
    
- Surreal colour relationships
    

However, when the objective is precise construction or geometry, literal language is generally easier to reason about.

### Engineering analogy

Think of the prompt as a specification.

Every additional phrase should ideally communicate one of the following:

- A required attribute
    
- A required relationship
    
- A compositional constraint
    
- A material property
    
- A lighting condition
    
- A stylistic intention
    

Words that contribute none of these may be unnecessary.

## 6. Our Café Portrait Experiment

Our practical experiments centred on a realistic café portrait generated with a Krea 2 Extended checkpoint.

The image included:

- An adult blonde woman
    
- Long, softly styled hair
    
- Emerald-green eyes
    
- Red lipstick
    
- A charcoal ribbed turtleneck
    
- A short pleated skirt
    
- A leather belt and accessories
    
- Dark tights and knee-high boots
    
- A white coffee cup
    
- A dim café environment
    

The character evolved through repeated prompting and refinement.

### What worked well?

Krea 2 produced convincing results for several aspects of the scene.

These included:

- Realistic clothing textures
    
- Material definition
    
- Facial features
    
- Lighting
    
- Skin rendering
    
- Hair
    
- Jewellery
    
- Environmental atmosphere
    

The model responded usefully to natural-language descriptions of physical details.

### What required additional work?

Some details were difficult to obtain consistently.

The eyes were the most obvious example.

We ultimately addressed those through inpainting and automated detailing rather than relying entirely on prompt revisions.

See [[ComfyUI — Automated Detailing]].

### Important lesson

Prompt engineering is only one part of the image-generation process.

A well-written prompt does not guarantee that every detail will be rendered correctly.

Some problems are better addressed through targeted refinement.

## 7. Prompting for Materials

Material descriptions can strongly influence how an object appears.

Consider a sweater.

A generic description:

_Dark sweater._

A more specific description:

_A fitted charcoal wool turtleneck with fine vertical ribbing and a soft, matte knitted texture._

The second description communicates:

- Colour
    
- Material
    
- Garment type
    
- Fit
    
- Surface texture
    
- Reflective behaviour
    

### Why material specificity matters

Different materials interact with light differently.

For example:

|Material|Useful visual properties|
|---|---|
|Leather|Surface grain, flexibility, sheen|
|Wool|Fibres, softness, knitted texture|
|Silk|Smoothness, drape, reflective highlights|
|Denim|Woven texture, seams, stiffness|
|Metal|Specular reflections, hard edges|
|Ceramic|Smooth surface, glaze, reflected light|
|Lace|Fine patterned structure, transparency|

A model may infer these properties from the material name alone.

Additional description becomes useful when the precise appearance matters.

### Avoid over-specification

Describing every material property in exhaustive detail can overwhelm the prompt.

Prioritise the features that distinguish the intended object from plausible alternatives.

## 8. Prompting for Spatial Relationships

Spatial relationships are especially important for complex scenes.

A prompt containing all the correct objects can still produce the wrong composition if their relationships are ambiguous.

### Example: A gothic mansion courtyard

A tag-based description might include:

`gothic mansion, courtyard, pathway, iron gates, moonlight, mist`

This identifies the scene elements.

However, it does not establish the spatial layout.

A more explicit description:

_A broad, level gothic courtyard extends from the open iron gates toward the mansion. A stone pathway runs across the courtyard toward the main entrance, with the mansion standing beyond it._

This communicates:

- The courtyard is level.
    
- The pathway crosses the courtyard.
    
- The gates and mansion occupy different ends of the scene.
    
- The mansion is the destination of the pathway.
    

### Our earlier architectural experiments

When developing the Chateau Falco setting, we encountered repeated problems with the model interpreting a pathway as a steeply rising approach toward the building.

The required correction was not simply another object tag.

It was a clearer description of perspective, ground level and spatial relationships.

### Lesson

When a model repeatedly generates the correct objects in the wrong arrangement, investigate the spatial specification before adding more descriptive attributes.

## 9. Prompting for Composition

Composition determines how the scene is arranged within the image.

Important considerations include:

- Camera position
    
- Viewing angle
    
- Subject placement
    
- Framing
    
- Perspective
    
- Foreground and background relationships
    
- Leading lines
    
- Depth
    
- Negative space
    

### Subject placement

If the prompt merely says:

_A woman standing in a corridor._

The model has considerable freedom to decide where she appears.

A more constrained description might say:

_The woman stands slightly to the right of the frame, with the corridor extending diagonally into the background._

This introduces a deliberate compositional relationship.

### Rule of thirds

A model should not be expected to follow the rule of thirds merely because the prompt describes a realistic scene.

If off-centre composition matters, it should be specified.

### Perspective

Perspective descriptions are especially important for architectural scenes.

Examples include:

- Eye-level camera
    
- Low-angle view
    
- Level ground plane
    
- Strong one-point perspective
    
- Parallel architectural verticals
    
- Pathway receding toward a distant entrance
    

These provide more useful geometric guidance than generic phrases such as _cinematic composition_.

## 10. Prompting for Facial Expression

Facial expression is another area where natural language can provide greater precision.

Tag-based descriptions might include:

`smile, parted lips, looking at viewer`

These communicate useful concepts but may leave considerable ambiguity.

Natural language can specify the relationship between features.

For example:

_Her lips are slightly parted in a restrained smile, with relaxed cheeks and a direct, confident gaze._

This describes:

- Mouth position
    
- Expression intensity
    
- Cheek behaviour
    
- Eye direction
    
- Overall emotional impression
    

### Avoid contradictory expressions

A prompt that simultaneously requests a broad smile, tightly closed lips, neutral expression and visible teeth may introduce conflicting requirements.

Where possible, describe a coherent facial expression rather than stacking unrelated expression tags.

### Our eye-refinement experience

Even when the prompt successfully established green eyes and a forward gaze, the model occasionally produced malformed eye anatomy.

This illustrates the difference between:

**Semantic correctness:** the image contains the requested feature.

**Anatomical correctness:** the feature is rendered plausibly.

Prompting can influence both, but localised refinement may still be required.

## 11. Krea 2 vs Illustrious

Our experience with Krea 2 changed how we approached realistic image generation.

### Illustrious

Illustrious and its derivatives are strongly associated with Danbooru-style prompting and anime-oriented image-generation ecosystems.

Tag-based prompts are therefore a natural starting point.

Our earlier Illustrious workflows also used model-specific techniques, including quality tags and detailer-related trigger words.

### Krea 2

In our experiments, Krea 2 was more rewarding when we used descriptive natural language to specify realistic details and relationships.

We found it particularly effective for:

- Realistic materials
    
- Clothing construction
    
- Lighting
    
- Skin and hair
    
- Physical interactions
    
- Environmental descriptions
    

### Important qualification

These observations describe our experience with the particular checkpoints and workflows we tested.

They do not establish that every Krea 2 model is universally superior to every Illustrious checkpoint for photorealism.

Checkpoint training, model architecture, prompt style, LoRAs and workflow configuration all influence the result.

### Practical conclusion

Use prompting conventions that suit the model.

Do not assume that a prompt optimised for one architecture will be interpreted identically by another.

## 12. Our Danbooru-to-Natural-Language Workflow

The approach we developed can be expressed as a sequence of transformations.

```
Character or Scene Concept
            |
            v
      Danbooru Tags
            |
            v
     Visual Inventory
            |
            v
  Natural-Language Expansion
            |
            v
   Relationship Refinement
            |
            v
      Krea 2 Prompt
            |
            v
      Base Generation
            |
            v
    Optional Detailing
```

### Stage 1 — Establish the visual inventory

Begin with tags describing:

- Subject
    
- Physical appearance
    
- Clothing
    
- Accessories
    
- Pose
    
- Environment
    
- Lighting
    
- Composition
    

The objective is to identify what should exist in the image.

### Stage 2 — Group related concepts

Organise the tags into meaningful categories.

For example:

**Character**

Blonde hair, green eyes, adult woman.

**Clothing**

Sweater, pleated skirt, leather belt, boots.

**Pose**

Sitting, crossed legs, holding coffee cup.

**Environment**

Dim café, table, warm interior lighting.

### Stage 3 — Expand into natural language

Convert the inventory into coherent descriptive language.

Preserve the intended attributes while making relationships explicit.

### Stage 4 — Resolve ambiguity

Identify details that require greater precision.

Examples:

- How the belt attaches to the skirt
    
- How the cup is held
    
- Where the character sits
    
- How the clothing fits
    
- Where the light originates
    
- How the background is arranged
    

### Stage 5 — Generate and inspect

Run the prompt through the base Krea 2 workflow.

Inspect the result for:

- Missing attributes
    
- Incorrect relationships
    
- Unwanted interpretations
    
- Anatomical problems
    
- Material inconsistencies
    
- Composition errors
    

### Stage 6 — Refine deliberately

Change the prompt where the problem is semantic or compositional.

Use detailing or inpainting where the problem is localised rendering quality.

This prevents unnecessary prompt complexity.

## 13. Prompt Enhancement and LLM Assistance

Our Krea 2 workflow also explored prompt enhancement using a language model.

The purpose was to transform user-provided text or tags into a more detailed prompt.

### Potential benefits

An enhancer can:

- Expand shorthand descriptions
    
- Group related attributes
    
- Express relationships more explicitly
    
- Improve grammatical consistency
    
- Produce reusable prompt structures
    

### Potential risks

An enhancer can also:

- Invent details
    
- Change the intended character
    
- Introduce unwanted stylistic assumptions
    
- Over-specify irrelevant features
    
- Replace precise requirements with decorative prose
    

### Faithfulness matters

A useful prompt enhancer should preserve the user's intent.

It should not treat every prompt as an invitation to redesign the scene.

For example, a request for a black crew-neck shirt should not become an elaborate costume simply because the enhancer considers it more visually interesting.

### Our workflow experiments

We successfully used Qwen3-VL within our Krea 2 workflow for prompt-related processing.

We also found that enabling thinking in the relevant template improved the stability of its text-generation behaviour.

That was an observed workflow result, not proof that every Qwen3-VL configuration requires the same setting.

### Practical principle

Treat an enhanced prompt as a proposed specification.

Review it before assuming that every generated detail reflects the original intent.

## 14. Controlled Prompt Experiments

Prompt engineering benefits from controlled experimentation.

Our workflow used a fixed seed to help compare changes.

### Why fix the seed?

A fixed seed reduces one source of variation.

This makes it easier to investigate the effect of changing a prompt.

For example:

**Experiment A**

Charcoal ribbed turtleneck.

**Experiment B**

Soft black fitted blouse.

If the model and other generation settings remain unchanged, comparing the outputs can reveal how the clothing description affects the result.

### Important limitation

Changing a prompt can influence more than the named attribute.

A clothing change may also affect:

- Pose
    
- Lighting
    
- Body shape
    
- Colour balance
    
- Facial appearance
    
- Composition
    

A fixed seed does not isolate a semantic attribute perfectly.

### Recommended experimental method

1. Preserve the working prompt.
    
2. Fix the seed and generation settings.
    
3. Change one intended attribute or relationship.
    
4. Generate the new result.
    
5. Compare the complete image.
    
6. Keep or reject the change based on the actual result.
    

This approach provides clearer evidence than changing several unrelated prompt elements simultaneously.

## 15. Character Identity and Prompt Drift

Our café portrait evolved considerably during experimentation.

The original objective was to generate a particular character.

However, as we refined clothing, facial features, accessories and realism, the resulting image gradually developed a different identity.

This was not necessarily a technical failure.

It was a consequence of exploratory generation and changing artistic decisions.

### Why character drift occurs

A prompt does not normally establish a persistent character identity.

Even with a fixed seed, changes to the prompt can alter:

- Facial structure
    
- Hair
    
- Expression
    
- Apparent age
    
- Clothing
    
- Body proportions
    
- Overall personality
    

### Character specification vs image refinement

These are different activities.

**Character specification** determines who the character should be.

**Image refinement** improves how a particular interpretation is rendered.

A technically excellent image can still depict the wrong character.

### Our practical lesson

We successfully improved the image's visual quality, particularly its eyes and realistic details.

However, the final character was no longer quite the person originally imagined.

This is a useful reminder to distinguish technical image quality from fidelity to the intended character.

## 16. Clothing and Personality

Clothing does more than establish appearance.

It also influences the viewer's interpretation of a character.

Our café portrait eventually reached a point where the heavy charcoal turtleneck and vivid red lipstick no longer suited the personality we had in mind.

Both details were technically well rendered.

The issue was artistic suitability.

### Why this matters

A prompt can be technically successful while being creatively wrong.

For example:

- A polished turtleneck may suggest restraint or sophistication.
    
- Bright red lipstick may suggest a more deliberate, glamorous presentation.
    
- A relaxed blouse and muted lip colour may communicate a different personality.
    

These associations are contextual rather than universal.

### Revising the character

Once the rendering pipeline was stable, we could return to artistic decisions.

Possible changes included:

- Replacing the sweater
    
- Softening the lipstick
    
- Adjusting accessories
    
- Revising expression
    
- Changing the overall styling
    

The important shift was that we no longer needed to solve every technical problem before experimenting with character design.

## 17. When to Use LoRAs Instead of Prompting

Some visual requirements are difficult to achieve reliably through text alone.

LoRAs can provide additional learned information that influences the model's output.

Potential applications include:

- Specific character appearances
    
- Clothing styles
    
- Artistic styles
    
- Repeated visual motifs
    
- Particular objects or details
    

### Prompting vs LoRA

|Requirement|Prompting|LoRA|
|---|---|---|
|General clothing colour|Usually appropriate|Often unnecessary|
|Simple pose description|Useful starting point|May not be required|
|Specific recurring character|May be inconsistent|Compatible character LoRA may help|
|Distinctive visual style|Can influence style|Compatible style LoRA may help|
|Complex specialised object|May be unreliable|Suitable trained LoRA may help|
|Exact spatial arrangement|Descriptive prompting helps|LoRA alone does not guarantee geometry|

### Compatibility matters

A LoRA must be compatible with the intended model architecture and loading implementation.

A LoRA trained for Illustrious cannot simply be assumed to work correctly with Krea 2.

### Our architectural preference

Rather than continually changing entire checkpoints to obtain individual details, we began exploring compatible LoRAs as more targeted additions to a stable base model.

This preserves the established workflow while allowing specialised visual influences.

See [[Krea 2 — Practical Workflow Architecture]].

## 18. Common Prompting Problems

|Symptom|Possible cause|Better investigation|
|---|---|---|
|Correct objects, wrong arrangement|Relationships underspecified|Describe geometry and placement|
|Clothing looks implausible|Construction underspecified|Explain how components connect|
|Character looks different|Prompt drift|Compare changed attributes|
|Image is overly stylised|Excessive style language|Simplify stylistic instructions|
|Material looks wrong|Ambiguous material description|Specify texture and surface properties|
|Subject always centred|Composition unconstrained|Specify placement and framing|
|Anatomy remains malformed|Local rendering limitation|Consider targeted detailing|
|Prompt enhancer changes intent|Unfaithful expansion|Review generated prompt|
|Too many competing details|Conflicting or overloaded instructions|Prioritise essential requirements|
|Model ignores a detail|Weak or ambiguous conditioning|Rephrase, simplify or use compatible specialised tools|

These are diagnostic starting points rather than guaranteed explanations.

## 19. A Practical Prompt-Construction Checklist

Before generating an image, consider the following.

### Subject

- Is the intended subject clearly identified?
    
- Are the important physical attributes specified?
    
- Is the character's apparent age and presentation appropriate?
    

### Clothing and objects

- Are the required items identified?
    
- Are important materials described?
    
- Are complex construction details explained?
    

### Relationships

- Are interactions between objects clear?
    
- Are spatial relationships explicit?
    
- Are poses physically coherent?
    

### Composition

- Is the camera position defined where necessary?
    
- Is subject placement intentional?
    
- Are foreground and background relationships clear?
    

### Lighting and atmosphere

- Is the light source or direction relevant?
    
- Is the intended mood communicated?
    
- Are material reflections consistent with the scene?
    

### Prompt quality

- Does every major phrase contribute useful information?
    
- Are there contradictory instructions?
    
- Has unnecessary decorative language been removed?
    

### Refinement strategy

- Is the problem actually a prompting problem?
    
- Would localised inpainting be more appropriate?
    
- Would automated detailing help?
    
- Is a compatible LoRA justified?
    

## 20. Key Lessons

### 1. Danbooru and natural language are complementary

Tags provide an efficient visual inventory.

Natural language provides richer control over relationships and construction.

### 2. Precision matters more than verbosity

Additional words are useful when they resolve ambiguity.

Longer prompts are not automatically better.

### 3. Describe relationships, not merely objects

A belt, skirt and belt loops are separate concepts.

Explaining how they connect communicates the intended construction.

### 4. Composition requires deliberate instruction

Do not assume that a model will apply a particular framing or perspective unless it is specified or strongly implied.

### 5. Prompting cannot solve every rendering problem

Localised anatomical defects may be better addressed through inpainting or detailing.

### 6. Fixed seeds support controlled experiments

They improve repeatability but do not guarantee character identity or perfectly isolated changes.

### 7. Technical quality and artistic suitability are different

A beautifully rendered image can still depict the wrong character.

### 8. Model-specific prompting conventions matter

Danbooru-style prompting remains useful for Illustrious, while our Krea 2 experiments rewarded precise natural-language descriptions.

### 9. Prompt enhancement should preserve intent

An enhancer is useful when it clarifies the specification, not when it silently redesigns it.

### 10. Use the simplest appropriate tool

Prompting, LoRAs, inpainting and automated detailing solve different problems.

Choose according to the actual failure or creative requirement.

---

## Related Notes

- [[ComfyUI — Understanding Latent Space]]
    
- [[ComfyUI — Noise Seeds and Reproducibility]]
    
- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Automated Detailing]]
    
- [[ComfyUI — Switches and Execution Flow]]
    
- [[ComfyUI — Subgraph Architecture]]
    
- [[Krea 2 — Practical Workflow Architecture]]
    

## Origin

Developed from practical experiments with Krea 2 Extended and Illustrious-derived image-generation workflows in ComfyUI Desktop.

The central experiment involved generating and refining a realistic café portrait, including clothing construction, material appearance, facial features, accessories and composition.

The work demonstrated that Danbooru-style tags remain an effective starting point for describing visual content, while natural-language prompting provides additional control over physical relationships and scene construction.

Further experiments with prompt enhancement, fixed seeds, manual inpainting and automated detailing reinforced the distinction between prompt interpretation, model rendering behaviour and workflow processing.

The resulting approach treats prompting as **visual specification rather than decorative prose**, using the level of linguistic precision appropriate to each requirement.