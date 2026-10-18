---
type: reference
status: active
created: 2026-10-18
source:
tags:
  - ai-filmmaking
  - comfyui
  - wan-2-2
  - image-to-video
  - shot-construction
  - continuity
  - storyboarding
  - compositing
  - video-editing
---
> [!abstract] Overview  
> This article describes a practical, shot-based approach to producing short films using AI-generated images and video.
> 
> It combines character design, environment plates, compositing, image-to-video generation, continuity management and nonlinear editing into a coherent production pipeline.
> 
> The central case study is our Nate Temple sequence at Chateau Falco, developed using Animij/Anima for still-image generation and WAN 2.2 for animation.
> 
> **Core principle: AI should improvise the performance. It shouldn't improvise the continuity.**

---

## 1. AI Video Generation Is Not the Same as Filmmaking

An image-to-video model can generate movement from a still image.

It may produce convincing:

- Character animation.
    
- Facial expressions.
    
- Clothing movement.
    
- Environmental motion.
    
- Camera movement.
    
- Lighting changes.
    
- Visual effects.
    

However, producing an attractive video clip is not equivalent to producing a coherent film.

A film requires relationships between shots.

Those relationships include:

- Character identity.
    
- Location.
    
- Spatial orientation.
    
- Cause and effect.
    
- Action.
    
- Timing.
    
- Emotional progression.
    
- Visual continuity.
    

The challenge is therefore not simply to generate better individual clips.

It is to construct a sequence in which the clips work together.

### 1.1 The change in approach

Our early experiments concentrated on persuading WAN to generate a complete character action.

As the project developed, we shifted toward designing shorter, more controlled shots and assembling them through editing.

This reduces the amount of narrative responsibility assigned to any single generation.

---

## 2. The Shot as the Fundamental Production Unit

A shot is a continuous piece of visual action presented from a particular camera viewpoint.

For AI filmmaking, each shot should have a clearly defined purpose.

A useful shot specification identifies:

|Property|Question|
|---|---|
|Subject|Who or what is visible?|
|Location|Where does the shot take place?|
|Starting state|What is true at the first frame?|
|Action|What changes during the shot?|
|Camera|How is the scene framed or filmed?|
|Duration|How long must the action remain usable?|
|Ending state|What must be true at the cut?|
|Continuity|How does this connect to adjacent shots?|

The clearer these properties are, the easier it becomes to evaluate a generated result.

### 2.1 Avoid unnecessary action complexity

A shot does not need to contain an entire sequence of movements.

For example:

**Complex shot:**

Nate crouches, gathers magic, looks upward, stands, turns, raises his hands and unleashes an explosion.

**Simpler shot:**

Nate remains crouched while unstable magical energy gathers around his hands.

The second version asks the model to maintain fewer simultaneous constraints.

Subsequent actions can be handled in separate shots.

### 2.2 Short shots are not a compromise

Short shots are a normal part of filmmaking.

They can improve:

- Pacing.
    
- Dramatic emphasis.
    
- Audience attention.
    
- Character reactions.
    
- Visual continuity.
    
- Editorial flexibility.
    

A three-second shot may be more useful than a five-second clip containing two seconds of unwanted motion.

---

## 3. The Production Pipeline

Our preferred architecture separates image creation, shot preparation, animation and editing.

```
Character and environment design
                |
                v
       Canonical visual assets
                |
                v
         Shot construction
                |
                v
         Approved keyframe
                |
                v
          WAN 2.2 I2V
                |
                v
       Generated video takes
                |
                v
       Review and selection
                |
                v
        Nonlinear editing
                |
                v
         Completed sequence
```

### 3.1 Still-image generation

Animij, an Anima-family checkpoint, is used to establish the visual appearance of the scene.

This stage is responsible for:

- Character appearance.
    
- Wardrobe.
    
- Environment.
    
- Composition.
    
- Starting pose.
    
- Initial lighting.
    
- Initial effects.
    

### 3.2 Image preparation

Inpainting and compositing correct or combine elements before animation.

This may include:

- Removing unwanted symbols.
    
- Reconstructing obscured architecture.
    
- Combining character and environment plates.
    
- Correcting perspective.
    
- Integrating shadows.
    
- Matching lighting.
    

### 3.3 Video generation

WAN 2.2 I2V animates the approved starting image.

Its primary responsibility is to generate motion.

### 3.4 Editing

The nonlinear editor determines which generated moments become part of the film.

It controls:

- Shot order.
    
- Timing.
    
- Transitions.
    
- Pacing.
    
- Reaction shots.
    
- Sound.
    
- Final colour treatment.
    

**The editor—not the video-generation model—has final authority over the sequence.**

---

## 4. Canonical Assets and Visual Continuity

AI models may generate plausible but inconsistent interpretations of the same character or environment.

A canonical asset provides an approved visual reference.

For our project, important canonical assets include:

- Nate's character design.
    
- Chateau Falco's architecture.
    
- Nate's standard wardrobe.
    
- The visual character of his wild magic.
    
- Approved starting keyframes.
    

### 4.1 Character continuity

Nate's established design includes:

- Adult male.
    
- Tall, athletic build.
    
- Short, messy dark hair.
    
- Light stubble.
    
- Confident, slightly cocky expression.
    
- Plain black crew-neck shirt.
    
- Open black coat.
    
- Dark trousers.
    
- Leather boots.
    

The shirt must remain free of the unwanted cross-like emblem that appeared during early generations.

### 4.2 Environment continuity

Chateau Falco should retain recognisable gothic architectural characteristics.

Important features include:

- Dark weathered masonry.
    
- Steep gothic rooflines.
    
- Tall gables.
    
- Towers and spires.
    
- Pointed architectural openings.
    
- A broad stone courtyard.
    
- Distinctive blue twilight and warm illumination.
    

Falco should not transform into an unrelated stately home, cathedral or generic fantasy castle between shots.

### 4.3 Effect continuity

Nate's magic should have a consistent visual identity.

Our preferred interpretation combines:

- Orange-gold energy.
    
- Blue and violet energy.
    
- Unstable, irregular motion.
    
- Contrasting illumination.
    
- Escalating intensity.
    

The colour variation is intentional.

A single uniform magical colour can make the effect appear more controlled than the character's wild magic should feel.

See [[ComfyUI — Environment Plates and LoRA Strategy]].

---

## 5. Approved Keyframes

An approved keyframe is the still image used to establish a shot's starting visual state.

It should be treated as a production asset rather than a disposable prompt result.

### 5.1 Why keyframe approval matters

If a source image contains an incorrect feature, WAN may preserve or amplify it.

Examples include:

- Wrong clothing.
    
- Unwanted symbols.
    
- Incorrect architecture.
    
- Inconsistent character appearance.
    
- Bad hand anatomy.
    
- Implausible ground contact.
    

These are generally easier to correct in a still image than across an animated sequence.

### 5.2 Keyframe approval checklist

Before passing an image to WAN, confirm:

- Character identity is correct.
    
- Wardrobe matches the intended shot.
    
- Environment matches the established location.
    
- Starting pose is appropriate.
    
- Camera framing supports the action.
    
- Lighting is coherent.
    
- Hands and visible anatomy are acceptable.
    
- No unwanted symbols or duplicate characters are present.
    
- The image supports the intended motion.
    
- The composition matches the planned edit.
    

### 5.3 Approved does not mean perfect

Not every minor imperfection requires correction.

The important question is whether the defect affects:

- Identity.
    
- Continuity.
    
- Visual credibility.
    
- The intended action.
    
- The final edit.
    

A small imperfection outside the visible crop may be irrelevant.

An incorrect character face is not.

---

## 6. Designing Shots for Image-to-Video

A useful I2V shot begins with a starting state that naturally supports the intended action.

### 6.1 Start close to the required action

If the shot requires Nate standing with magical energy in both hands, begin with Nate already standing.

Do not require the model to transition from a crouch unless the transition itself is important.

Similarly, if the shot requires Alucard to put on sunglasses, begin with an image in which the glasses and hands are positioned appropriately for that action.

### 6.2 Avoid unnecessary hidden transformations

A model may struggle when an action reveals previously unseen details.

For example, standing from a crouch may require inventing:

- Leg proportions.
    
- Coat length.
    
- Previously hidden clothing.
    
- Boot positions.
    
- Body orientation.
    

These inventions can cause identity and wardrobe drift.

A separately generated standing keyframe avoids much of this uncertainty.

### 6.3 Give each shot one dominant action

A shot may contain incidental motion, but it should have a clear primary purpose.

Examples:

- Nate gathers magic.
    
- Alucard puts on sunglasses.
    
- Carl laughs.
    
- Gunter facepalms.
    
- Nate's eyes ignite.
    

This makes success easier to recognise and failures easier to diagnose.

---

## 7. Case Study — The Nate Temple Sequence

Our planned sequence takes place outside Chateau Falco at night.

The central idea is that Nate is preparing something dangerous and probably spectacularly ill-advised.

The other characters react according to their personalities.

### Planned shot sequence

|Shot|Subject|Primary action|Narrative purpose|
|---|---|---|---|
|1|Nate|Crouches and gathers unstable magic|Establish impending danger|
|2|Alucard|Puts on sunglasses|Confident anticipation|
|3|Carl|Giggles|Comic escalation|
|4|Gunter|Facepalms|Exasperated recognition|
|5|Nate|Stands with magic, eyes blaze|Climactic reveal|

The sequence uses reaction shots to create anticipation before returning to Nate.

This is not merely a technical workaround.

It is also an effective comedic and dramatic structure.

---

## 8. Shot 1 — Nate Prepares the Magic

### 8.1 Starting image

Our approved Scene 1 illustration showed Nate crouching in Falco's courtyard.

The image established:

- Nate's appearance.
    
- Black coat and shirt.
    
- Gothic environment.
    
- Contrasting magical colours.
    
- Dramatic lighting.
    
- A strong initial composition.
    

The unwanted chest emblem was removed through inpainting.

The corrected still became:

**Scene 1 Master — CLEAN**

### 8.2 Intended motion

The desired animation was restrained.

Nate should remain crouched while magical energy develops.

Subtle motion might include:

- Movement in the coat.
    
- Energy flickering.
    
- Hand movement.
    
- Small changes in expression.
    
- Environmental light reacting to magic.
    

### 8.3 Actual WAN behaviour

The native WAN generation produced approximately three useful seconds.

After that, Nate began standing and turning his head.

His expression became more aggressive than intended.

The camera also introduced a stronger push-in than seen in the Turbo version.

### 8.4 Prompt experiments

We tried both detailed and concise motion instructions intended to preserve the crouched pose.

The unwanted transition persisted.

This demonstrated a limitation of the tested generation rather than proving that the prompt was inherently wrong.

### 8.5 Production decision

Use the successful portion of the clip.

Do not insist on retaining all generated frames.

The remainder can be discarded during editing.

**The accepted result is a usable shot, not necessarily a usable complete generation.**

---

## 9. Reaction Shots as Continuity Tools

Reaction shots show how other characters respond to an event.

They serve several purposes:

- Establish character personality.
    
- Build anticipation.
    
- Control pacing.
    
- Introduce humour.
    
- Redirect audience attention.
    
- Conceal changes between principal shots.
    

### 9.1 Alucard

Alucard putting on sunglasses communicates that he anticipates what is coming.

The action is visually simple and characterful.

The glasses also provide a clear focal point for the shot.

### 9.2 Carl

Carl's giggle adds an escalating sense of mischief.

His reaction suggests that Nate's impending action is entertaining rather than reassuring.

### 9.3 Gunter

Gunter's facepalm communicates exasperation.

The audience understands that this is probably not the first time Nate has done something catastrophically reckless.

### 9.4 Why the reactions matter technically

While these shots play, the audience is no longer watching Nate's body position.

This creates an opportunity to cut from:

**Nate crouched**

to:

**Nate standing**

without requiring a continuous generated transition.

The audience infers that he stood during the intervening reactions.

This is a conventional editing technique applied to an AI-generation limitation.

---

## 10. Shot 5 — The Magical Reveal

### 10.1 Starting state

Nate is already standing in Falco's courtyard.

He wears the established black outfit.

His hands contain relatively restrained magical energy.

His eyes initially appear normal.

### 10.2 Intended progression

The shot develops toward a clear climax:

1. The magical energy becomes more active.
    
2. Orange-gold and blue-violet effects intensify.
    
3. The surrounding light responds.
    
4. Nate's eyes begin to blaze.
    
5. The shot cuts at the moment of maximum dramatic impact.
    

### 10.3 Why begin with restrained magic?

If the starting keyframe already contains maximum intensity, the video model has less room to create visible escalation.

Beginning with smaller effects gives the shot a clearer visual progression.

### 10.4 Why cut at the climax?

Continuing beyond the climax introduces opportunities for:

- Unwanted pose changes.
    
- Excessive effects.
    
- Character distortion.
    
- Loss of visual focus.
    
- Unnecessary anticlimax.
    

A deliberate cut can be stronger than allowing the model to continue improvising.

---

## 11. Native Versus Turbo Generation

Our WAN 2.2 workflow supports native and LightX2V Turbo modes.

### Recorded configuration

|Setting|Native|Turbo|
|---|---|---|
|Steps|20|4|
|CFG|3.5|1.0|
|High/low split|10|2|
|LightX2V LoRAs|Disabled|Enabled|

### 11.1 Turbo's production role

Turbo is useful for:

- Testing basic motion ideas.
    
- Checking whether a prompt is understood.
    
- Evaluating approximate action.
    
- Rapidly rejecting unsuitable concepts.
    
- Testing workflow connectivity.
    

### 11.2 Native's production role

Native generation is used when evaluating the intended final performance.

However, native is not simply a higher-quality rendering of the Turbo result.

In our experiments, it produced different:

- Camera motion.
    
- Facial expressions.
    
- Character movement.
    

### 11.3 Practical implication

Turbo can help decide whether an idea is worth exploring.

It cannot guarantee the precise native result.

The native output must be reviewed independently.

See [[WAN 2.2 — ComfyUI Workflow Architecture]].

---

## 12. Generation Duration Versus Editorial Duration

A five-second generation does not have to become a five-second shot.

This distinction is central to efficient AI filmmaking.

### 12.1 Generation duration

The amount of video produced by WAN.

### 12.2 Usable duration

The portion of that generation that satisfies the shot requirements.

### 12.3 Editorial duration

The portion actually used in the finished sequence.

For example:

|Stage|Duration|
|---|---|
|Generated clip|~5 seconds|
|Visually acceptable portion|~3 seconds|
|Final edited shot|Determined in editing|

These values describe our first Nate experiment, not a universal expected ratio.

### 12.4 Selective use of generated footage

A clip should be evaluated frame by frame or over short intervals.

Look for:

- Identity drift.
    
- Unwanted camera movement.
    
- Anatomical changes.
    
- Clothing changes.
    
- Inconsistent effects.
    
- Unintended action.
    
- Loss of continuity.
    

A clip can be valuable even if only part of it is usable.

---

## 13. Continuity Across Cuts

Continuity allows the audience to understand how separate shots relate to one another.

AI filmmaking requires explicit attention to continuity because each generation may independently reinterpret the scene.

### 13.1 Character continuity

Track:

- Appearance.
    
- Wardrobe.
    
- Props.
    
- Position.
    
- Pose.
    
- Expression.
    
- Current action.
    
- Visible effects.
    

### 13.2 Spatial continuity

Track:

- Character positions.
    
- Camera direction.
    
- Building orientation.
    
- Ground geometry.
    
- Relative distances.
    
- Direction of movement.
    

### 13.3 Lighting continuity

Track:

- Time of day.
    
- Ambient colour.
    
- Key light direction.
    
- Magical illumination.
    
- Window lighting.
    
- Shadow relationships.
    

### 13.4 Temporal continuity

Track what has already happened.

For example:

- Nate begins crouched.
    
- The other characters react.
    
- Nate is standing when the camera returns.
    
- His magic intensifies.
    
- His eyes ignite.
    

Each shot should be compatible with the implied passage of time.

---

## 14. The 180-Degree Rule and Screen Direction

The 180-degree rule is a traditional filmmaking principle used to preserve spatial orientation.

An imaginary axis is established between subjects or along the direction of an action.

Keeping cameras on one side of that axis generally helps preserve consistent screen direction.

### 14.1 Why this matters for AI-generated shots

Different generations may place characters and architectural features in inconsistent positions.

For example:

- Falco's entrance appears on the left in one shot.
    
- It appears on the right in the next.
    
- Nate's position relative to the entrance changes unexpectedly.
    

Such changes may confuse the audience.

### 14.2 Practical approach

Before generating a shot, establish:

- The location of the main action.
    
- The direction characters face.
    
- The camera's side of the action.
    
- Major background landmarks.
    

Not every shot must obey the 180-degree rule.

Crossing the axis can be deliberate.

The important distinction is between intentional cinematography and accidental inconsistency.

---

## 15. Camera Movement as a Controlled Variable

WAN may introduce camera motion even when it is not central to the prompt.

Our native Nate experiment demonstrated this through an unexpected push-in.

### 15.1 Common camera behaviours

Potential camera movements include:

- Push-in.
    
- Pull-back.
    
- Pan.
    
- Tilt.
    
- Orbit.
    
- Tracking movement.
    
- Handheld-style motion.
    

### 15.2 Camera movement affects continuity

An unexpected push-in may alter:

- Framing.
    
- Visible architecture.
    
- Character scale.
    
- Editing options.
    
- Relationship to adjacent shots.
    

### 15.3 Production approach

Specify the intended camera behaviour when it matters.

For a reaction shot, a relatively stable camera may be preferable.

For a magical reveal, a deliberate push-in may enhance the drama.

However, text conditioning does not guarantee exact camera control.

The generated result must still be evaluated.

---

## 16. Building a Shot Specification

Before generating video, document the intended shot.

A reusable specification can be relatively compact.

### Example — Scene 5

|Field|Specification|
|---|---|
|Shot ID|Scene 5 — Nate reveal|
|Location|Chateau Falco courtyard|
|Subject|Nate Temple|
|Starting pose|Standing|
|Wardrobe|Plain black shirt, open black coat, dark trousers, boots|
|Starting effects|Small orange-gold and blue-violet magical orbs|
|Action|Magic intensifies and eyes ignite|
|Camera|Deliberate framing, consistent with preceding shots|
|Ending|Cut at peak magical intensity|
|Continuity|Nate has stood during the intervening reaction shots|

This is a production specification, not a complete model prompt.

The generation prompt should be developed separately for the chosen model and workflow.

### 16.1 Why specifications help

They separate creative intent from model-specific syntax.

The same shot specification can inform:

- Still-image prompting.
    
- Inpainting.
    
- Compositing.
    
- WAN motion prompting.
    
- Editorial decisions.
    

This makes it easier to revise the technical implementation without losing the intended scene.

---

## 17. Reviewing Generated Takes

A generated clip should be treated as a take.

It may be:

- Accepted.
    
- Partially accepted.
    
- Rejected.
    
- Used as a reference for another attempt.
    

### 17.1 Review criteria

|Category|Questions|
|---|---|
|Identity|Does the character remain recognisable?|
|Action|Does the intended action occur?|
|Camera|Is the movement appropriate?|
|Continuity|Does the shot fit the sequence?|
|Effects|Are the effects coherent and correctly coloured?|
|Anatomy|Are body movements plausible?|
|Environment|Does the location remain stable?|
|Timing|Is the useful action long enough?|
|Editing|Is there a clean entry and exit point?|

### 17.2 Do not judge only the final frame

A clip may begin successfully and deteriorate later.

Conversely, an early awkward movement may be followed by a useful section.

Review the full temporal progression.

### 17.3 Preserve useful partial takes

Even a rejected full clip may contain valuable material.

A short reaction, expression or motion fragment may be useful elsewhere.

However, do not preserve footage merely because generation was expensive.

Editorial quality takes priority over sunk computational cost.

---

## 18. Nonlinear Editing as a Generative-AI Control Layer

The nonlinear editor provides control that the diffusion model does not.

It allows us to determine:

- Exact shot duration.
    
- Cut points.
    
- Sequence order.
    
- Rhythm.
    
- Sound synchronisation.
    
- Transitions.
    
- Final colour consistency.
    

### 18.1 Editing can solve generation problems

Consider the original Nate problem.

WAN would not reliably maintain the crouched pose for the entire requested clip.

Possible responses included:

- Repeated prompt changes.
    
- More generation attempts.
    
- Different seeds.
    
- Further workflow experiments.
    

Instead, we selected the successful portion and redesigned the sequence around it.

This was an editorial solution.

### 18.2 Editing can also improve storytelling

The reaction shots do more than conceal the pose transition.

They establish personality and anticipation.

Alucard's sunglasses, Carl's giggle and Gunter's facepalm collectively tell the audience that something ridiculous is about to happen.

The sequence becomes more entertaining because of the edit.

---

## 19. Sound Design and Timing

Sound is not yet a completed part of our documented Nate/Falco production workflow, but it will be important to the finished sequence.

Potential sound elements include:

- Ambient night atmosphere.
    
- Wind through the courtyard.
    
- Magical energy.
    
- Clothing movement.
    
- Sunglasses handling.
    
- Carl's laughter.
    
- Gunter's exasperated reaction.
    
- A rising magical crescendo.
    
- A final impact or abrupt cut.
    

### 19.1 Sound reinforces continuity

Consistent ambience can help separate shots feel like one location.

For example, a continuous background wind track can connect:

- Nate's preparation.
    
- Alucard's reaction.
    
- Carl's reaction.
    
- Gunter's reaction.
    
- Nate's reveal.
    

### 19.2 Sound controls anticipation

The magical sound can build across the reaction shots even while Nate is off-screen.

This implies that his activity continues during the cuts.

The audience hears the escalation before seeing the final result.

### 19.3 Editing to sound

The final reveal may benefit from synchronising:

- Eye ignition.
    
- Magical energy peak.
    
- Sound crescendo.
    
- Cut to black or the next shot.
    

These are creative options to explore, not established final decisions.

---

## 20. Shot-Based Generation Efficiency

AI video generation can be computationally expensive.

Our recorded WAN experiments included:

|Mode|Resolution|Duration|Runtime|
|---|---|---|---|
|Native|640×640|5 seconds|~10m 40s|
|Native|448×640|5 seconds|~5m 51s|
|Turbo|640×640|5 seconds|~96s|
|Turbo|320×448|5 seconds|~20s|

These are local measurements from our RTX 5070 Ti workflow.

### 20.1 Reduce unnecessary generation

Before generating, determine:

- What the shot needs to accomplish.
    
- How much usable duration is required.
    
- Whether Turbo can answer the current question.
    
- Whether the source image is already approved.
    
- Whether a previous take contains useful material.
    

### 20.2 Avoid endless prompt refinement

When repeated prompt changes fail to correct the same behaviour, reconsider the shot design.

The issue may be:

- An unsuitable starting pose.
    
- Too much requested motion.
    
- Insufficient visual conditioning.
    
- An overly long action.
    
- A continuity requirement better handled by editing.
    

### 20.3 Use the right tool for the problem

|Problem|Preferred area of investigation|
|---|---|
|Wrong face|Still-image generation / character conditioning|
|Wrong clothing|Inpainting / character plate|
|Wrong architecture|Environment plate|
|Bad perspective|Geometry / compositing|
|Incorrect starting pose|Keyframe generation|
|Unwanted motion|WAN prompting / shot design|
|Unusable ending|Trim or regenerate|
|Pose change between shots|Editing / reaction shots|
|Inconsistent exposure|Colour grading|
|Poor pacing|Nonlinear editing|

The goal is not to solve every problem inside WAN.

---

## 21. Production Asset Management

As the project expands, generated assets should remain traceable.

A conceptual structure might be:

```
AI-Film/
│
├── Characters/
│   ├── Nate/
│   ├── Alucard/
│   ├── Carl/
│   └── Gunter/
│
├── Environments/
│   └── Chateau-Falco/
│
├── Storyboards/
│
├── Keyframes/
│   ├── Scene-01/
│   └── Scene-05/
│
├── Video-Takes/
│   ├── Scene-01/
│   └── Scene-05/
│
├── Audio/
│
├── Editing/
│
└── Exports/
```

This is a proposed structure, not a claim about directories already created.

### 21.1 Useful metadata

For each video take, record:

- Shot identifier.
    
- Source keyframe.
    
- Workflow version.
    
- Model files.
    
- Prompt.
    
- Seed.
    
- Sampling configuration.
    
- Resolution.
    
- Frame count.
    
- Generation runtime.
    
- Review status.
    
- Usable time range.
    

### 21.2 Why metadata matters

Without generation metadata, reproducing a useful take becomes difficult.

Metadata also supports controlled comparisons.

For example, two clips may appear different because of:

- A changed prompt.
    
- A different seed.
    
- Native versus Turbo sampling.
    
- A changed source image.
    
- A different resolution.
    

Recording these variables makes the differences easier to interpret.

---

## 22. Continuity Tracking

For larger projects, continuity should be documented explicitly.

A simple continuity record can include:

|Category|Example|
|---|---|
|Location|Chateau Falco courtyard|
|Time|Night|
|Character|Nate Temple|
|Wardrobe|Black shirt and open black coat|
|Initial state|Crouched|
|Intermediate state|Standing during reaction sequence|
|Final state|Standing with intensified magic|
|Magic colours|Orange-gold, blue-violet|
|Architecture|Canonical Falco|
|Lighting|Cool ambient twilight with magical illumination|

This record provides a reference when generating later shots.

### 22.1 Continuity is cumulative

Each accepted shot establishes additional facts.

Once the audience has seen a character holding an object in one hand, the next shot should not casually move it to the other.

Once the courtyard layout is established, later shots should respect it unless the camera viewpoint justifies the change.

The more shots a sequence contains, the more valuable explicit continuity tracking becomes.

---

## 23. Practical Shot Production Procedure

### Step 1 — Define the narrative purpose

Write down what the shot contributes to the sequence.

### Step 2 — Define the starting and ending states

Identify what must be visible at the beginning and what should change before the cut.

### Step 3 — Select canonical references

Choose the appropriate character and environment assets.

### Step 4 — Generate the starting image

Use the established still-image workflow and compatible LoRAs where appropriate.

### Step 5 — Correct the image

Use inpainting or compositing for identity, wardrobe, environment and perspective corrections.

### Step 6 — Approve the keyframe

Do not proceed until continuity-critical elements are acceptable.

### Step 7 — Generate an exploratory video

Use the appropriate WAN configuration.

Turbo may be useful for inexpensive concept testing.

### Step 8 — Generate a production take

Use the intended final sampling configuration.

### Step 9 — Review the complete clip

Identify the useful section and any continuity failures.

### Step 10 — Select the edit

Trim the shot to the strongest usable duration.

### Step 11 — Integrate with adjacent shots

Check pacing, spatial continuity, action continuity and visual consistency.

### Step 12 — Apply finishing

Complete sound design, grading and final export as appropriate.

---

## 24. Lessons from the Nate/Falco Experiments

### Lesson 1 — The starting image is a production decision

WAN should not be responsible for establishing every continuity-critical detail.

Approve the image before animation.

### Lesson 2 — One clear action is easier to control

Short, focused shots reduce opportunities for unwanted motion.

### Lesson 3 — Generated duration is not editorial duration

Use the best frames, not necessarily all frames.

### Lesson 4 — Reaction shots are valuable

They establish character, improve pacing and conceal transitions.

### Lesson 5 — Continuity can be solved through editing

Not every transition needs to be generated continuously.

### Lesson 6 — Native and Turbo are different configurations

Turbo results do not guarantee native camera movement or performance.

### Lesson 7 — Reusable environments improve consistency

Falco should be an established location, not a newly invented mansion in every shot.

### Lesson 8 — Character identity should be controlled upstream

Character LoRAs and canonical references are more appropriate for identity than relying on repeated video prompting alone.

### Lesson 9 — The model can improvise within boundaries

Allow natural motion and expression while retaining control over narrative and visual continuity.

### Lesson 10 — Filmmaking principles remain relevant

Framing, cutting, pacing, screen direction, reaction shots and sound design are just as important with AI-generated footage as with conventional footage.

---

## 25. The Production Philosophy

Our approach can be summarised as four layers.

**Design** establishes what the world and characters look like.

**Generation** creates images and motion within that established world.

**Selection** determines which results are acceptable.

**Editing** turns those results into a coherent story.

```
DESIGN
  |
  v
GENERATE
  |
  v
SELECT
  |
  v
EDIT
  |
  v
FILM
```

The creative objective is not to eliminate all model improvisation.

Some unexpected movement or expression may improve a shot.

The objective is to decide where improvisation is useful and where consistency is essential.

> [!quote] Guiding principle  
> **AI should improvise the performance. It shouldn't improvise the continuity.**

---

## 26. Related Notes

- [[WAN 2.2 — Video Generation Fundamentals]] — Video diffusion, I2V and motion generation.
    
- [[WAN 2.2 — ComfyUI Workflow Architecture]] — Dual-expert sampling and native/Turbo configurations.
    
- [[ComfyUI — Inpainting and Scene Geometry]] — Masks, perspective and environmental reconstruction.
    
- [[ComfyUI — Environment Plates and LoRA Strategy]] — Canonical characters, environments and compositing.
    

---

## 27. References

### Technical resources

- [WAN 2.2 — Official repository](https://github.com/Wan-Video/Wan2.2)
    
- [ComfyUI documentation](https://docs.comfy.org/)
    
- [ComfyUI source repository](https://github.com/comfyanonymous/ComfyUI)
    

### Project experiments

- Nate Scene 1 — Original illustration.
    
- Scene 1 Master — CLEAN.
    
- WAN 2.2 native and Turbo I2V experiments.
    
- Nate crouching motion experiments.
    
- Falco environment reconstruction.
    
- Planned five-shot Nate/Alucard/Carl/Gunter sequence.
    

The production techniques documented here combine established filmmaking concepts with observations from our local AI-generation experiments. Planned workflow improvements are distinguished from techniques already tested.