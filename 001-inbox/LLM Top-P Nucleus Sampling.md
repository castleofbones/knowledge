---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-sampling
  - top-p
  - temperature
  - sillytavern
  - inference-settings
purpose: knowledge
---
## Overview

**Top-P**, also known as **nucleus sampling**, is a token-sampling parameter that controls which candidate tokens are eligible for selection during language model generation.

Rather than limiting generation to a fixed number of tokens, Top-P dynamically selects the smallest set of the most probable tokens whose combined probability reaches or exceeds a specified threshold.

The central distinction is:

> **Temperature reshapes the probability distribution. Top-P determines how much of that distribution remains available for sampling.**

Top-P influences the diversity of generated text without directly changing the relative probabilities of the retained tokens. Those probabilities are subsequently renormalized for sampling.

## How Top-P Works

Consider a language model generating the next token after:

`The cat sat on the ...`

Suppose the model produces the following hypothetical probability distribution:

|Candidate token|Probability|Cumulative probability|
|---|---|---|
|mat|40%|40%|
|floor|25%|65%|
|chair|15%|80%|
|sofa|8%|88%|
|table|5%|93%|
|windowsill|3%|96%|
|refrigerator|1%|97%|
|Other tokens|3%|100%|

These probabilities are illustrative rather than actual model output.

### Example: Top-P = 0.80

The sampling process:

1. Sorts candidate tokens by descending probability.
    
2. Adds tokens to the candidate set in that order.
    
3. Calculates the cumulative probability after each addition.
    
4. Stops when the cumulative probability reaches or exceeds 80%.
    
5. Excludes all remaining tokens from sampling.
    
6. Renormalizes the probabilities of the retained candidates.
    

The resulting candidate set is:

|Token|Original probability|Renormalized probability|
|---|---|---|
|mat|40%|50%|
|floor|25%|31.25%|
|chair|15%|18.75%|

Only these three tokens can be selected.

Importantly, excluded tokens do not merely become less likely. **Their selection probability becomes zero.**

The retained tokens are still sampled probabilistically. Top-P does not automatically select the highest-probability token.

## Why Top-P Is Dynamic

Unlike Top-K, which limits sampling to a fixed number of candidate tokens, Top-P adapts to the model's confidence.

### High-confidence predictions

Consider:

`The capital of France is ...`

Suppose the model assigns:

- Paris: 99.7%
    
- All other candidates: 0.3%
    

With Top-P = 0.90, the candidate set contains only `Paris`.

The model is already confident, so the sampling pool is extremely narrow.

### Low-confidence predictions

Now consider:

`The mysterious stranger opened the door and saw ...`

Many continuations may be plausible.

The probability distribution could be spread across dozens or hundreds of candidate tokens.

With the same Top-P = 0.90, substantially more candidates may be retained.

**The number of eligible tokens changes according to the shape of the probability distribution.**

This adaptive behaviour is the principal advantage of nucleus sampling.

## Interaction with Temperature

Temperature and Top-P influence different stages of token selection.

### Temperature

Temperature transforms the model's probability distribution.

- Lower temperature concentrates probability around the most likely tokens.
    
- Higher temperature distributes probability more evenly across candidates.
    
- Temperature = 1.0 leaves the original softmax distribution unchanged.
    

### Top-P

Top-P filters the resulting probability distribution by cumulative probability.

- Lower Top-P retains a smaller proportion of probability mass.
    
- Higher Top-P retains more of the distribution.
    
- Top-P = 1.0 performs no nucleus filtering.
    

### Combined behaviour

|Temperature|Top-P|General effect|
|---|---|---|
|Low|Low|Strongly constrained, predictable output|
|Low|High|Highly probable tokens dominate despite a broad candidate pool|
|High|Low|Flatter distribution, but restricted to its highest-probability nucleus|
|High|High|Wider variety of candidates with more opportunity for unusual selections|
|Moderate|Moderate–high|A compromise between variation and predictability|

The interaction is important because changing temperature also changes how many tokens fall within a particular Top-P threshold.

For example, increasing temperature can flatten the probability distribution, meaning more tokens may be needed to accumulate 90% probability mass.

Consequently, raising temperature while keeping Top-P unchanged can increase both:

- The number of eligible candidate tokens.
    
- The probability that less conventional tokens are selected.
    

This is why simultaneously increasing both parameters can produce unexpectedly erratic output.

## What Top-P = 1.0 Actually Means

A common misconception is that Top-P = 1.0 represents maximum creativity.

It does not.

**Top-P = 1.0 means that no tokens are excluded by nucleus sampling.**

Temperature still determines their relative selection probabilities.

For example, a token with an extremely small probability remains extremely unlikely to be selected when temperature is low, even if Top-P allows it into the candidate set.

Top-P controls eligibility, not creativity directly.

## Practical Implications for SillyTavern Roleplay

For character-driven roleplay, Top-P influences how broadly the model can explore alternative continuations.

### Lower Top-P

Potential benefits:

- More conventional phrasing.
    
- Greater predictability.
    
- Fewer low-probability token selections.
    

Potential drawbacks:

- Repetitive or formulaic dialogue.
    
- Reduced linguistic variety.
    
- Less opportunity for unexpected but contextually appropriate responses.
    

### Higher Top-P

Potential benefits:

- Greater variety in dialogue and descriptions.
    
- More unusual but potentially appropriate continuations.
    
- Less aggressive removal of alternative wording.
    

Potential drawbacks:

- Increased exposure to unlikely continuations.
    
- Greater risk of incoherent output when combined with high temperature.
    
- Less predictable character dialogue.
    

These are tendencies, not guarantees. Model behaviour also depends on training, prompting, context and other sampler settings.

### Suggested tuning approach

For a local model used through SillyTavern:

1. Establish a working baseline with Temperature = 1.0 and Top-P = 1.0.
    
2. Adjust temperature to establish the desired degree of variation.
    
3. Reduce Top-P gradually if the model produces too many unlikely or incoherent continuations.
    
4. Compare outputs using the same prompt and similar context.
    
5. Avoid changing multiple sampling parameters simultaneously during initial tuning.
    

These values are diagnostic starting points, not verified optimal settings for any particular model.

Other samplers, including Top-K and Min-P, can further restrict the eligible candidate pool. Their interactions depend on the backend and its sampler implementation.

## Important Implementation Details

### Threshold crossing

Top-P normally includes the token that causes cumulative probability to reach or exceed the threshold.

For example, if cumulative probability rises from 75% to 85%, a Top-P threshold of 0.80 generally retains the token responsible for reaching 85%.

The retained probability mass may therefore exceed the requested threshold.

### Renormalization

After filtering, the retained probabilities are normalized so their sum equals 100%.

This is why a token originally assigned 40% probability can have a 50% selection probability after filtering.

### Token-by-token operation

Top-P is applied independently at each token-generation step.

The model recalculates the next-token distribution after every generated token, so the nucleus changes continuously throughout a response.

### Backend differences

Actual behaviour may differ between inference engines because of:

- Sampler ordering.
    
- Minimum candidate-retention rules.
    
- Interactions with Top-K, Min-P and repetition penalties.
    
- Backend-specific defaults and implementation details.
    

Do not assume two inference engines produce identical results merely because their Top-P settings match.

## Key Takeaways

- **Top-P is a probability-mass threshold, not a fixed candidate count.**
    
- Candidate tokens are ranked by probability and retained until their cumulative probability reaches the threshold.
    
- Excluded tokens cannot be selected.
    
- Retained token probabilities are renormalized before sampling.
    
- The number of eligible tokens adapts to the model's confidence.
    
- Temperature changes the distribution; Top-P filters that distribution.
    
- Top-P = 1.0 disables nucleus filtering but does not guarantee creative output.
    
- Temperature and Top-P should be tuned together carefully, ideally changing one parameter at a time.
    

**Mental model:** Temperature changes the shape of the probability landscape. Top-P draws a boundary around the most probable part of that landscape and excludes everything outside it.