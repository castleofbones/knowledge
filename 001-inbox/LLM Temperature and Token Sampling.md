---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-sampling
  - temperature
  - sillytavern
  - inference-settings
---
## Overview

**Temperature controls how strongly a language model favours high-probability tokens over less probable alternatives during text generation.**

It is commonly described as a creativity setting:

- Lower temperature produces more predictable output.
    
- Higher temperature produces more varied output.
    

Although useful as a starting point, this description hides the underlying mechanism.

**Temperature does not give a model more ideas. It changes the probability of selecting less-likely tokens from the possibilities the model has already calculated.**

This distinction is particularly relevant when configuring local language models for creative writing and roleplay in SillyTavern.

## How Token Selection Works

A language model generates text by predicting the next token based on the preceding context.

Consider the incomplete sentence:

> Nate stepped through the doorway and...

The model might assign the following probabilities to possible continuations:

|Token|Original probability|
|---|---|
|looked|35%|
|saw|25%|
|stopped|18%|
|froze|10%|
|grinned|7%|
|exploded|1%|
|danced|0.3%|

These values are illustrative, and the table shows only a subset of the possible tokens.

The model has already evaluated these possibilities before temperature is applied.

Temperature changes their relative probabilities before the next token is sampled.

## How Temperature Changes Probabilities

Temperature modifies the probability distribution using the following transformation:

[  
P_i(T) = \frac{P_i^{1/T}}{\sum_j P_j^{1/T}}  
]

Where:

- (P_i) is the original probability of token (i).
    
- (T) is the temperature.
    
- (P_i(T)) is the adjusted probability.
    

This transformation is mathematically equivalent to dividing the model's logits by temperature before applying softmax.

### Temperature Below 1

A temperature below 1 sharpens the probability distribution.

High-probability tokens become more dominant, while unlikely alternatives become even less likely.

For example:

> Nate entered the room and looked around.

The model tends towards conventional, statistically probable continuations.

This generally improves predictability but can reduce variation.

### Temperature Equal to 1

At temperature 1, the original probability distribution is preserved.

No sharpening or flattening occurs.

This does not mean that generation becomes deterministic. Tokens are still sampled according to their probabilities unless another decoding method changes that behaviour.

### Temperature Above 1

A temperature above 1 flattens the probability distribution.

Less-likely tokens become more competitive with the original favourites.

For example:

> Nate swaggered through the doorway, took one look at the wreckage, and decided—with impressive optimism—that none of it was technically his fault.

The continuation is less conventional and potentially more expressive.

However, increasing temperature does not improve the model's underlying reasoning, vocabulary, knowledge or creativity.

It simply increases the likelihood of selecting less probable continuations.

## Illustrative Temperature Levels

The following examples demonstrate the general effect rather than exact or universal temperature settings.

|Temperature|Typical behaviour|
|---|---|
|0|Greedy or near-deterministic selection, depending on implementation|
|0.3|Strong preference for highly probable tokens|
|0.7|Moderate variation while retaining strong probability preferences|
|1.0|Original model probability distribution|
|1.2|Greater opportunity for unusual continuations|
|Very high|Increasingly unpredictable and potentially incoherent output|

These are not fixed quality thresholds. The effects depend on the model, prompt, other sampling parameters and inference backend.

At an excessively high temperature, a continuation might resemble:

> Nate swaggered through the doorway, moonlight arguing violently with Tuesday as the wallpaper remembered fire.

This illustrates an important limitation: **increased randomness is not equivalent to increased creativity or quality.**

## Why High Temperature Can Cause Output Drift

Language models generate text sequentially.

Each selected token becomes part of the context used to predict the next token.

The process is approximately:

```
Existing context
      |
      v
Predict next-token probabilities
      |
      v
Apply temperature
      |
      v
Apply other sampling controls
      |
      v
Select next token
      |
      v
Append token to context
      |
      +----> Repeat
```

A relatively unlikely token may still produce a coherent continuation.

However, selecting an unusual token changes the context for every subsequent prediction.

At higher temperatures, further unlikely choices become more probable.

This can produce a compounding effect:

1. An unexpected token changes the direction of the sentence.
    
2. The model predicts continuations based on that new direction.
    
3. Another unlikely token introduces additional deviation.
    
4. The generated text may progressively move away from the original intention.
    

This explains why excessive temperature can cause incoherence, narrative drift or unusual word combinations.

It is not simply that individual words become random. The entire generated continuation can follow an increasingly unlikely trajectory.

## Temperature Versus Creativity

Temperature is best understood as a control over **sampling diversity**, not creativity itself.

A model's ability to produce compelling creative writing also depends on:

- Its training and learned writing capabilities.
    
- The quality and specificity of the prompt.
    
- The available context.
    
- Character definitions and narrative constraints.
    
- Other sampling parameters.
    
- The model's ability to maintain coherence across long responses.
    

A low temperature can still produce excellent creative writing when the prompt and model strongly support it.

A high temperature can produce repetitive, incoherent or uninspired writing.

**The objective is not to maximise temperature, but to find a balance between variation and coherence.**

## Interaction With Other Sampling Controls

Temperature operates alongside other decoding parameters.

### Top-P

Top-P, also called nucleus sampling, limits token selection to a set of high-probability candidates whose cumulative probability reaches a configured threshold.

Temperature changes the relative probabilities; Top-P restricts the candidate pool.

The two therefore influence different aspects of token selection.

Their combined effect depends on the implementation and order in which sampling transformations are applied.

### Top-K

Top-K restricts sampling to a maximum number of the highest-ranked candidate tokens.

Temperature can change their relative probabilities, but it cannot restore tokens excluded by Top-K.

### Min-P

Min-P filters candidates according to their probability relative to the most probable token.

It can prevent extremely unlikely tokens from being selected, even when temperature increases their relative probability.

### Repetition Penalty

Repetition penalties modify token preferences based on previously generated text.

They address repetition rather than directly controlling sampling diversity.

Their interaction with temperature can influence how readily a model repeats familiar phrases or chooses alternatives.

## Practical Implications for SillyTavern Roleplay

For roleplay and creative dialogue, temperature influences how readily a model departs from conventional continuations.

Lower temperatures may favour:

- Consistent phrasing.
    
- Predictable dialogue.
    
- Conservative narrative developments.
    
- Stronger preference for common expressions.
    

Higher temperatures may encourage:

- Less conventional dialogue.
    
- More surprising descriptions.
    
- Unexpected character reactions.
    
- Greater stylistic variation.
    

However, excessively high settings can also introduce:

- Inconsistent character behaviour.
    
- Narrative drift.
    
- Illogical developments.
    
- Strange vocabulary choices.
    
- Reduced coherence.
    

There is no universally optimal temperature for roleplay.

Different models can respond quite differently to identical settings.

### Suggested Testing Method

To evaluate temperature for a particular model:

1. Select a fixed roleplay prompt and character context.
    
2. Keep the model and other sampling parameters unchanged.
    
3. Generate multiple responses at each temperature.
    
4. Compare coherence, character consistency, narrative variation and writing quality.
    
5. Repeat with different scenes to avoid overfitting to one prompt.
    
6. Select a temperature that provides useful variation without sacrificing consistency.
    

Using multiple generations per setting is important because sampling is probabilistic. One response cannot reliably establish how a temperature behaves.

These are recommended experiments, not documented test results.

## Temperature Versus Maximum Response Tokens

Temperature and maximum response tokens control fundamentally different things.

**Temperature determines how probabilities are distributed among candidate tokens.**

**Maximum response tokens determines how many tokens the model is permitted to generate.**

A useful analogy:

- Temperature influences how adventurous the model is when choosing a fork in the road.
    
- Maximum response tokens determines how far it is allowed to travel.
    

Increasing the output-token limit does not increase creativity.

Increasing temperature does not increase the maximum response length.

## Key Takeaways

1. Temperature transforms the probability distribution used for next-token sampling.
    
2. Lower temperatures strengthen the preference for likely tokens.
    
3. Higher temperatures increase the relative probability of unlikely tokens.
    
4. Temperature 1 preserves the original distribution.
    
5. Temperature does not create new knowledge, ideas or capabilities.
    
6. Unusual token choices can compound because each generated token influences subsequent predictions.
    
7. Excessive temperature can reduce coherence rather than improve creativity.
    
8. Temperature must be considered alongside Top-P, Top-K, Min-P and other sampling controls.
    
9. Optimal settings are model-dependent and should be evaluated experimentally.
    

**The central principle: Temperature controls how willing the model is to choose less-likely continuations, not how creative the model fundamentally is.**