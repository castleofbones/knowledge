---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-sampling
  - repetition-penalty
  - sillytavern
  - text-generation
  - inference-settings
---
## Overview

**Repetition Penalty** is a text-generation parameter that discourages a Large Language Model (LLM) from repeatedly selecting tokens that have already appeared in its context.

Its purpose is to reduce repetitive wording, recurring phrases, and generation loops while preserving the model's ability to reuse words when appropriate.

The fundamental principle is:

> A token that has already appeared becomes less attractive, but is not necessarily excluded from selection.

Repetition Penalty is particularly relevant to conversational AI, creative writing, and roleplay, where excessive repetition can make generated responses unnatural.

## How Repetition Penalty Works

An LLM generates text by predicting the probability of possible next tokens.

Before the final token is sampled, Repetition Penalty modifies the scores (logits) of tokens that have already appeared within the configured repetition window.

This changes their relative likelihood of being selected.

### Example Without Repetition Penalty

Consider the partial sentence:

> The wizard walked into the room and looked at the...

The following probabilities are illustrative rather than actual model measurements.

|Token|Original Probability|
|---|---|
|wizard|24%|
|door|21%|
|table|17%|
|room|14%|
|window|10%|
|llama|0.001%|

Both `wizard` and `room` have already appeared in the sentence.

Without a repetition penalty, these tokens remain eligible according to their original probabilities.

### Example With Repetition Penalty

After applying a repetition penalty, the distribution might resemble:

|Token|Adjusted Probability|
|---|---|
|door|28%|
|table|23%|
|window|16%|
|wizard|13%|
|room|8%|

The previously used tokens have become less likely, allowing alternatives to become more competitive.

**Important:** These probabilities are illustrative. The actual results depend on the model, original logits, penalty implementation, and other sampling parameters.

## Understanding the Penalty Value

In the commonly used multiplicative repetition-penalty implementation:

|Value|Behaviour|
|---|---|
|`1.0`|Neutral; no repetition penalty|
|`1.05`|Very mild discouragement|
|`1.10`|Moderate discouragement|
|`1.15`|Stronger discouragement|
|`1.20`|Potentially excessive for natural conversation|
|Below `1.0`|May encourage repetition|

These descriptions are qualitative, not universal thresholds.

A value of `1.10` does not simply subtract 10% from a token's probability.

In a common implementation, the adjustment is applied to logits before probability normalization:

```
if token_was_previously_used:
    if logit > 0:
        logit = logit / repetition_penalty
    else:
        logit = logit * repetition_penalty
```

This preserves the intended discouragement for both positive and negative logits.

The resulting probabilities depend on the complete distribution.

Different inference engines may implement repetition controls differently.

## Why Repetition Penalty Is Necessary

Autoregressive language models generate each token using the context produced so far.

This creates the possibility of self-reinforcing repetition.

For example:

> He looked at the door.  
> He looked at the door.  
> He looked at the door.  
> He looked at the door.

Once a repeated sequence becomes highly probable, the model may continue reproducing it.

Repetition Penalty reduces the likelihood of reusing tokens from that sequence, potentially allowing generation to escape the loop.

It does not guarantee that repetition loops will be prevented.

## Problems With Excessive Repetition Penalty

A high repetition penalty can damage the quality of generated text.

The model may begin avoiding words that are entirely appropriate to repeat, including:

- Character names
    
- Pronouns
    
- Common grammatical words
    
- Technical terminology
    
- Important locations and objects
    

### Example: Excessive Synonym Substitution

Instead of:

> Nate entered the room. Nate examined the doorway before returning to the table.

The model might generate:

> Nate entered the room. The sorcerer surveyed the chamber. The magician approached the doorway. The spellcaster returned to the furniture.

Although the second passage avoids repeated terminology, it is less natural and may introduce unwanted descriptions.

This is particularly problematic in roleplay, where consistent names, terminology, and character identities matter.

**The objective is to prevent undesirable repetition, not eliminate repetition altogether.**

## Repetition Penalty Range

Some inference engines and SillyTavern-compatible backends expose an additional setting commonly called **Repetition Penalty Range** or **Repeat Last N**.

This controls how many previous tokens are examined when identifying repetition.

For example:

```
Repetition Penalty: 1.10
Repetition Range:   256
```

Conceptually, this instructs the sampler to penalize tokens appearing within the most recent 256 tokens.

Tokens outside that window are no longer penalized solely because they appeared earlier.

### Short Versus Long Ranges

|Range|Potential Effect|
|---|---|
|Short|Targets immediate repetition while allowing earlier vocabulary to return|
|Medium|Discourages repetition across a larger passage|
|Long|Can discourage reuse of vocabulary across substantial amounts of text|

A longer range is not automatically better.

For long-form roleplay, excessive range combined with a strong penalty may discourage the repeated use of character names and established terminology.

The meaning of special values such as `0` or `-1` is backend-dependent and should be verified against the actual inference engine.

## Interaction With Other Sampling Parameters

Repetition Penalty works alongside other generation settings but serves a different purpose.

|Parameter|Primary Function|
|---|---|
|Temperature|Changes the sharpness of the token probability distribution|
|Top-P|Retains the smallest group of tokens whose cumulative probability reaches a threshold|
|Top-K|Restricts sampling to a maximum number of highest-ranked tokens|
|Min-P|Excludes tokens whose probability falls below a threshold relative to the most probable token|
|Repetition Penalty|Discourages tokens already present in the configured context|

Temperature and probability-filtering controls influence the selection of candidate tokens.

Repetition Penalty instead modifies token scores based on previous usage.

The precise processing order depends on the inference backend.

Consequently, the same nominal sampling settings may behave differently across engines.

## Practical Guidance for SillyTavern Roleplay

For conversational and roleplay generation, a conservative repetition penalty is generally preferable.

A reasonable experimental starting point is:

```
Repetition Penalty: 1.05–1.10
```

This is a tuning suggestion, not a verified optimal configuration for a particular model.

### Recommended Testing Procedure

1. Start with a neutral penalty of `1.0`.
    
2. Generate several responses using a consistent prompt and sampling configuration.
    
3. Identify whether actual repetition problems occur.
    
4. Increase the penalty gradually, for example to `1.05`.
    
5. Compare repetition, naturalness, character-name consistency, and vocabulary.
    
6. Adjust the repetition range separately if the backend supports it.
    

Avoid changing Temperature, Top-P, Top-K, and Repetition Penalty simultaneously, because doing so makes the cause of any improvement difficult to identify.

### Symptoms and Possible Adjustments

|Symptom|Possible Adjustment|
|---|---|
|Repeated sentences or phrases|Increase penalty slightly|
|Repetitive vocabulary|Test a mild increase|
|Excessive synonyms|Reduce penalty|
|Avoidance of character names|Reduce penalty or shorten range|
|Unnatural sentence construction|Test a lower penalty|
|Persistent loops despite penalty|Investigate prompting, model behaviour, context, and other sampling settings|

Repetition Penalty should not be treated as a universal solution to repetitive output. Prompt construction, model characteristics, context quality, and sampling interactions can also contribute.

## Key Takeaways

- Repetition Penalty discourages previously used tokens rather than banning them.
    
- `1.0` is neutral in the common implementation.
    
- Values above `1.0` progressively discourage repetition.
    
- Excessive penalties can make prose unnatural.
    
- Repetition range determines how far back the sampler looks, where supported.
    
- The penalty modifies token scores, not probabilities by a fixed percentage.
    
- Conservative settings are generally preferable for natural conversational and roleplay output.
    
- Backend-specific behaviour should be verified before treating a configuration as definitive.
    

**In essence:** Repetition Penalty provides a limited form of short-term pressure against repeating previously generated vocabulary. Its usefulness comes from balancing variation against the legitimate need to reuse words.