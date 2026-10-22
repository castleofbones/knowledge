---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - min-p
  - temperature
  - sillytavern
  - token-sampling
  - inference-settings
purpose: knowledge
---
## Overview

**`Min-P` (Minimum Probability Sampling)** is a token filtering technique used during Large Language Model (LLM) text generation.

Unlike `Top-K`, which retains a fixed number of candidate tokens, or Top-P, which retains tokens based on cumulative probability, Min-P evaluates each candidate **relative to the probability of the most likely token**.

This produces a dynamic filtering threshold that adapts to the model's probability distribution at each generation step.

`Min-P` is particularly useful for creative writing and roleplay, where maintaining variation without allowing extremely unlikely token selections is desirable.

## 1. How Min-P Works

At each generation step, the model assigns probabilities to possible next tokens.

Min-P identifies the highest-probability token and calculates a minimum acceptable probability relative to that token.

The formula is:
$$ 
P_{\text{threshold}} = P_{\text{max}} \times \text{Min-P}  
$$
Where:

- ($P_{\text{max}}$) is the probability of the most likely token.
    
- `Min-P` is the configured sampling threshold.
    
- ($P_{\text{threshold}}$) is the minimum probability a token must have to remain eligible.
    

Tokens below this threshold are excluded from sampling.

### Example: Min-P = 0.10

Consider the following probability distribution:

|Token|Probability|
|---|---|
|cat|40%|
|dog|25%|
|fox|15%|
|llama|8%|
|kipper|5%|
|boing|4%|
|wibble|3%|

The most likely token is `cat`, with a probability of 40%.

With Min-P set to `0.10`:

```
40% * 0.10 = 4%  
```

Every token with a probability of at least 4% survives.

**Retained:** cat, dog, fox, llama, kipper, boing.

**Excluded:** wibble.

The remaining probabilities are renormalized before a token is selected, assuming no further filters are applied.

## 2. Understanding Min-P Values

One initially counterintuitive characteristic of Min-P is that **higher values produce more restrictive filtering**.

A value of `0.05` does not mean that the bottom 5% of the probability distribution is discarded.

It means that a token must have at least 5% of the probability of the leading token.

Assuming the leading token has a probability of 40%:

|Min-P|Probability cutoff|Filtering behaviour|
|---|---|---|
|0.01|0.4%|Very permissive|
|0.05|2%|Permissive|
|0.10|4%|Moderate|
|0.20|8%|Selective|
|0.50|20%|Highly selective|
|1.00|40%|Only tokens tied for highest probability|

### Candidate survival comparison

|Token|Probability|0.05|0.10|0.20|0.50|
|---|---|---|---|---|---|
|cat|40%|✓|✓|✓|✓|
|dog|25%|✓|✓|✓|✓|
|fox|15%|✓|✓|✓|✗|
|llama|8%|✓|✓|✓|✗|
|kipper|5%|✓|✓|✗|✗|
|boing|4%|✓|✓|✗|✗|
|wibble|3%|✓|✗|✗|✗|

**Key principle:** Increasing Min-P reduces the number of eligible tokens, while decreasing Min-P allows more candidates to participate.

This does not directly control the probability of choosing any individual token. It controls which tokens remain eligible.

## 3. Interaction Between Temperature and Min-P

Temperature modifies the shape of the token probability distribution.

- **Higher temperature** flattens the distribution, making lower-ranked tokens more competitive.
    
- **Lower temperature** sharpens the distribution, increasing the relative dominance of high-probability tokens.
    

Because Min-P calculates its threshold relative to the leading token, temperature can significantly influence how many candidates survive.

### 3.1 Baseline distribution

Using the original example