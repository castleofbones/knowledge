---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-sampling
  - presence-penalty
  - generation-settings
  - sillytavern
  - inference-settings
---
## Overview

**Presence Penalty** is a generation parameter that discourages a Large Language Model (LLM) from reusing tokens that have already appeared in the text being considered for repetition penalties.

Its primary purpose is to encourage variation in vocabulary and subject matter, reducing the tendency for a model to repeatedly return to the same words or concepts.

The fundamental principle is:

> Has this token appeared before? If so, reduce its likelihood of being selected again.

Unlike Frequency Penalty, Presence Penalty does not increase according to the number of times a token has appeared. A token that has appeared once receives the same presence penalty as one that has appeared twenty times.

## How Presence Penalty Works

During generation, the model calculates a set of logits representing the relative likelihood of possible next tokens.

Presence Penalty modifies these logits before the final token is sampled.

A common implementation uses:

$$l'_i = l_i - \alpha_p \cdot I(c_i > 0)  $$

Where:

- ($l_i$) is the original logit for token (i).
    
- ($l'_i$) is the adjusted logit.
    
- ($\alpha_p$) is the configured Presence Penalty.
    
- (*c_i*) is the number of previous occurrences of the token.
    
- ($I(c_i > 0)$) equals 1 if the token has appeared and 0 otherwise.
    

This means:

- **Presence Penalty = 0:** No adjustment.
    
- **Positive Presence Penalty:** Previously used tokens become less likely.
    
- **Negative Presence Penalty:** Previously used tokens become more likely, where supported.
    

The penalty is applied to logits, not directly to probabilities. The final probabilities are recalculated through the sampling process.

Exact implementation and the history considered depend on the inference backend.

## Example: Token Selection

Imagine the model is choosing its next token.

|Token|Initial Probability|Previously Used|
|---|---|---|
|magic|30%|Yes|
|energy|25%|No|
|power|20%|No|
|magic's|10%|No|
|spell|8%|Yes|

With Presence Penalty set to zero, the original distribution remains unchanged.

Increasing Presence Penalty reduces the logits associated with `magic` and `spell`.

Consequently, previously unused tokens such as `energy` and `power` become relatively more attractive.

The exact resulting probabilities depend on the penalty strength, the original logits, and other sampling settings.

**Important:** Presence Penalty operates on tokens, not complete words or semantic concepts. Different tokenizations or alternative words may escape the penalty even when they express the same idea.

## Example: Roleplay Dialogue

Consider this repetitive passage:

> Nate raised his hand. Magic crackled around his fingers. The magic grew brighter. His magic surged across the courtyard.

Repeated use of _magic_ makes the prose monotonous.

With an appropriate Presence Penalty, the model may produce something closer to:

> Nate raised his hand. Magic crackled around his fingers. The energy grew brighter, violet sparks twisting through the spell before power surged across the courtyard.

The second version introduces vocabulary variation while preserving the original meaning.

This is an illustrative outcome, not a guaranteed transformation.

Presence Penalty does not understand that _energy_, _spell_, and _magic_ are semantically related. It simply changes the relative attractiveness of tokens based on previous usage.

## Presence Penalty vs Frequency Penalty

These parameters are closely related but solve slightly different problems.

|Characteristic|Presence Penalty|Frequency Penalty|
|---|---|---|
|Trigger|Token appeared previously|Token appeared previously|
|Penalty strength|Fixed per previously used token|Increases with occurrence count|
|Primary effect|Encourages novelty|Discourages excessive repetition|
|Main risk|Avoidance of useful vocabulary|Unnatural avoidance of frequently needed tokens|

### Frequency Penalty

A common implementation uses:

$$l'_i = l_i - \alpha_f c_i  $$

The more frequently a token appears, the larger the penalty.

### Presence Penalty

Presence Penalty instead uses a binary condition:

$$l'_i = l_i - \alpha_p I(c_i > 0) $$

The number of occurrences does not affect the penalty magnitude.

### Combined Behaviour

Where both parameters are supported, their adjustments may be combined:

$$l'_i = l_i - \alpha_p I(c_i > 0) - \alpha_f c_i  $$

This allows Presence Penalty to encourage new vocabulary while Frequency Penalty discourages excessive reuse of individual tokens.

## Application to Roleplay and Creative Writing

Presence Penalty can be useful when an LLM repeatedly:

- Uses the same descriptive vocabulary.
    
- Returns to previously established subjects.
    
- Repeats particular phrases or expressions.
    
- Produces monotonous narration.
    
- Reuses the same imagery across successive responses.
    

However, excessive Presence Penalty can damage narrative continuity.

### Potential Problems

A model may become less willing to reuse:

- Character names.
    
- Location names.
    
- Important objects.
    
- Pronouns and common grammatical tokens.
    
- Terminology essential to the scene.
    
- Words needed to maintain consistent descriptions.
    

For example, avoiding the name _Nate_ merely because it has appeared previously does not improve the writing.

The model may instead produce increasingly awkward synonyms, circumlocutions, or unnatural sentence structures.

This is particularly undesirable in long-running roleplay, where consistency is often more important than novelty.

## Recommended Configuration

For conversational AI and roleplay, a conservative approach is preferable.

|Presence Penalty|General Guidance|
|---|---|
|0|Disabled; useful baseline|
|0.1–0.3|Mild encouragement of variation|
|0.4–0.6|More noticeable novelty pressure|
|0.7+|Greater risk of awkward vocabulary avoidance|

These ranges are **illustrative tuning guidance**, not universally validated settings. Different models and backends may interpret the parameter differently.

For SillyTavern roleplay with a local model, a sensible experimental approach is:

1. Begin with Presence Penalty at `0`.
    
2. Generate several responses using representative prompts.
    
3. Identify whether repeated vocabulary or topics are genuinely problematic.
    
4. Increase Presence Penalty gradually if necessary.
    
5. Compare outputs for readability, continuity, and natural dialogue.
    
6. Avoid adjusting Presence and Frequency Penalty simultaneously during initial testing.
    

Some inference backends may ignore or not support Presence Penalty. Confirm that the selected SillyTavern connection actually forwards the parameter and that the backend implements it.

## Interaction with Other Generation Settings

Presence Penalty does not operate in isolation.

- **Temperature:** Controls how strongly differences between token logits influence the final probability distribution.
    
- **Top-P:** Restricts sampling to a cumulative probability mass.
    
- **Top-K:** Restricts sampling to a maximum number of high-probability candidates.
    
- **Min-P:** Excludes candidates below a probability threshold relative to the most likely token.
    
- **Frequency Penalty:** Adds occurrence-count-based discouragement where supported.
    
- **Repetition Penalty:** May apply a separate multiplicative logit adjustment, depending on the backend.
    

Presence Penalty changes the candidate logits, potentially altering which tokens survive subsequent sampling filters.

The exact order of these operations is implementation-dependent.

## Practical Conclusions

1. Presence Penalty discourages reuse based on whether a token has appeared, not how often.
    
2. Frequency Penalty scales its effect according to repetition count.
    
3. Presence Penalty encourages lexical variation but does not directly understand semantic novelty.
    
4. Excessive penalties can harm character consistency and natural prose.
    
5. For roleplay, start with zero or a modest positive value and increase only when repeated vocabulary is a demonstrated problem.
    
6. Backend support and implementation details must be verified before assuming a SillyTavern setting is effective.
    

**Mental model:**

> Presence Penalty: "You've already used that token. Consider something else."
> 
> Frequency Penalty: "You keep using that token. The more you repeat it, the less attractive it becomes."