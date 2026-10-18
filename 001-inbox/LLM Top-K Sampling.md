---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-sampling
  - top-k
  - top-p
  - temperature
  - sillytavern
  - inference-settings
---

## Overview

**Top-K sampling** is a token-selection mechanism that restricts an LLM to choosing from the K highest-probability tokens at each generation step.

Unlike Temperature, which changes the relative probabilities of tokens, or Top-P, which selects candidates according to cumulative probability, **Top-K imposes a fixed maximum number of eligible tokens**.

Its primary purpose is to prevent sampling from the low-probability tail of the model's predicted token distribution.

## How Top-K Works

At each generation step, the model calculates a probability distribution across its vocabulary, potentially containing tens or hundreds of thousands of tokens.

Most of these tokens have extremely low probabilities.

For example:

|Rank|Token|Probability|
|---|---|---|
|1|sensible|31%|
|2|plausible|18%|
|3|unusual|11%|
|20|strange|0.8%|
|50|peculiar|0.15%|
|51|bizarre|0.14%|
|150|llama|0.001%|
|180|kipper|0.0005%|
|195|boing|0.0002%|
|200|wibble|0.0001%|

_Illustrative probabilities, not a measured model output._

With `Top-K = 50`, only the 50 highest-ranked tokens remain eligible.

Every token ranked below 50 is excluded, regardless of its original probability.

The surviving probabilities are then renormalised so they sum to 100%, and the next token is sampled from this restricted distribution.

**Important:** Top-K does not choose the most probable token automatically. It restricts the candidate pool while retaining probabilistic sampling.

## Top-K vs Top-P

Both mechanisms restrict token selection, but they use different criteria.

|Property|Top-P|Top-K|
|---|---|---|
|Selection criterion|Cumulative probability|Probability ranking|
|Pool size|Dynamic|Fixed maximum|
|Adapts to model confidence|Yes|No|
|Main purpose|Retain a chosen probability mass|Limit candidate count|
|Typical setting|0.9|40–50|

### High-confidence predictions

Consider:

> The capital of France is ...

The model might assign an overwhelming probability to `Paris`.

With `Top-P = 0.9`, only a very small number of candidates might be needed to reach the cumulative probability threshold.

With `Top-K = 50` alone, up to 50 candidates remain eligible, even though most are unlikely to be useful.

### Creative predictions

Consider:

> The strange creature emerging from the mist looked like ...

There might be hundreds of plausible continuations.

Top-P could retain a relatively large candidate pool because the probability distribution is spread across many possibilities.

Top-K restricts that pool to its configured maximum, potentially excluding otherwise reasonable creative alternatives.

This demonstrates Top-K's principal limitation: **it does not adapt to the shape of the probability distribution**.

## Interaction Between Temperature, Top-P and Top-K

The three settings control different aspects of token selection.

### Temperature — Probability distribution

Temperature changes how strongly the model favours high-probability tokens.

- Lower values concentrate probability on the most likely candidates.
    
- Higher values flatten the distribution, giving less likely candidates a greater chance.
    
- Temperature does not directly impose a candidate-count limit.
    

### Top-P — Probability boundary

Top-P retains the smallest set of highest-probability tokens whose cumulative probability reaches the configured threshold.

For example, `Top-P = 0.9` retains approximately 90% of the probability mass before renormalisation.

The number of surviving tokens depends on the distribution.

### Top-K — Candidate-count boundary

Top-K restricts the selection to the K highest-ranked tokens.

For example, `Top-K = 50` permits no more than 50 candidates.

### Combined example

Suppose Temperature and Top-P leave 200 eligible candidates.

Applying `Top-K = 50` reduces the final pool to 50.

The remaining 150 candidates are excluded, including extremely unlikely possibilities such as `llama`, `kipper`, `boing`, and `wibble`.

Conversely, if Top-P leaves only 17 candidates, a Top-K limit of 50 does not reduce the pool further.

The conceptual model is:

```
Model logits
    |
    v
Temperature
Reshape relative probabilities
    |
    v
Top-P
Restrict cumulative probability mass
    |
    v
Top-K
Restrict maximum candidate count
    |
    v
Renormalisation
    |
    v
Random sampling
    |
    v
Next token
```

**Implementation caveat:** This diagram illustrates the combined effect, not a guaranteed execution order. Sampling implementations may apply Top-K and Top-P in different sequences. The exact behaviour depends on the inference backend.

## Why Use Both Top-P and Top-K?

Top-P already removes low-probability tokens, so Top-K can appear redundant.

However, their interaction provides an additional constraint.

Top-P controls how much probability mass remains available, while Top-K prevents the candidate pool from becoming excessively large.

For example:

|Top-P candidates|Top-K setting|Maximum surviving candidates|
|---|---|---|
|12|50|12|
|35|50|35|
|80|50|50|
|200|50|50|

This combination allows Top-P to adapt to confident predictions while Top-K provides a fixed upper bound when predictions are more uncertain.

However, Top-K can also remove valid creative possibilities. It is not necessarily beneficial to enable both filters aggressively.

## Implications for Creative Writing and Roleplay

In creative roleplay, less probable tokens can produce interesting descriptions, unexpected dialogue, and original narrative developments.

Excessively restrictive Top-K settings may reduce this variety.

Conversely, very permissive sampling can increase the chance of unusual or incoherent continuations.

A practical tuning approach is:

1. Establish a sensible Temperature.
    
2. Use Top-P to control the probability mass available for sampling.
    
3. Introduce Top-K only when an additional candidate-count restriction is desirable.
    
4. Compare results using consistent prompts and generation settings.
    
5. Avoid changing several sampling parameters simultaneously.
    

For SillyTavern and local inference backends such as Ollama, the actual effect also depends on which sampling controls the backend supports and how it combines them.

A value of `Top-K = 0` may mean disabled or unrestricted in some implementations, but this is not universal and should be checked against the backend.

## Key Takeaways

- **Temperature** determines how strongly probable tokens are favoured.
    
- **Top-P** retains candidates according to cumulative probability.
    
- **Top-K** limits the number of candidates eligible for selection.
    
- Top-K is a maximum pool size, not a guarantee that exactly K tokens will survive other filters.
    
- Top-K does not make the final selection deterministic.
    
- Combining Top-P and Top-K can eliminate extremely unlikely tokens that survive a relatively permissive Top-P threshold.
    
- Overly restrictive Top-K settings can suppress useful creative alternatives.
    

The simplest mental model:

> **Temperature:** How adventurous should the probabilities be?
> 
> **Top-P:** How much of the probability distribution should remain available?
> 
> **Top-K:** Regardless of that, how many candidates are we willing to consider?

Top-K is the final headcount restriction in this conceptual model: even if `wibble` technically survives the probability threshold, it cannot be selected when its rank falls outside the permitted candidate pool.