---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-sampling
  - frequency-penalty
  - sillytavern
  - roleplay
  - inference-settings
---
## Overview

**Frequency Penalty** is a generation parameter that discourages a Large Language Model (LLM) from repeatedly producing the same tokens.

Unlike Presence Penalty, which applies a penalty once a token has appeared, Frequency Penalty increases the penalty according to **how many times that token has already occurred**.

The purpose is to reduce repetitive vocabulary and encourage greater linguistic variety.

A useful mental model:

- **Presence Penalty:** Have I used this token before?
    
- **Frequency Penalty:** How many times have I used this token?
    

## How Frequency Penalty Works

During generation, the model assigns scores (logits) to candidate tokens before converting those scores into probabilities.

Frequency Penalty modifies those scores based on the number of previous occurrences of each token.

A common implementation follows:
$$  
L'_i = L_i - (\alpha_f \times C_i)  
$$
Where:

| Symbol     | Meaning                                     |
| ---------- | ------------------------------------------- |
| $L_i$      | Original logit for token $i$                |
| $L'_i$     | Adjusted logit                              |
| $\alpha_f$ | Configured frequency penalty                |
| $C_i$      | Number of previous occurrences of token $i$ |

The more frequently a token appears, the larger its penalty becomes.

### Example

Assume a frequency penalty of `0.5` and the model repeatedly uses the token `magic`.

|Previous occurrences|Applied penalty|
|---|---|
|0|0|
|1|-0.5|
|2|-1.0|
|3|-1.5|
|6|-3.0|

Each occurrence makes the token progressively less attractive during subsequent sampling.

This does not prohibit the token from appearing again. It simply reduces its probability relative to competing tokens.

The final outcome also depends on other sampling parameters, including Temperature and Top-P.

## Frequency Penalty vs Presence Penalty

Although both settings discourage repetition, they operate differently.

### Presence Penalty

Presence Penalty applies a fixed adjustment when a token has appeared at least once.

Repeated occurrences do not increase that adjustment.

Conceptually:

$$  
L'_i = L_i - (\alpha_p \times I(C_i > 0))  
$$

Where $I$ is an indicator function that equals 1 when the token has appeared and 0 otherwise.

### Frequency Penalty

Frequency Penalty scales with the number of occurrences.

Consequently, frequently repeated tokens receive progressively stronger penalties.

### Comparison

|Behaviour|Presence Penalty|Frequency Penalty|
|---|---|---|
|Penalizes previously used tokens|Yes|Yes|
|Penalty increases with repetition|No|Yes|
|Encourages new vocabulary|Yes|Yes|
|Specifically targets repeated wording|Indirectly|More directly|
|Can discourage legitimate repetition|Yes|Yes|

When both are supported, their adjustments may be combined:

$$  
L'_i = L_i - (\alpha_p \times I(C_i > 0))