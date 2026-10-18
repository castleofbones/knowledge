---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-sampling
  - temperature
  - top-p
  - sillytavern
  - inference-settings
---
## Overview

Temperature and Top-P are two sampling parameters that work together to control how a Large Language Model (LLM) selects its next token.

Although both influence output variation, they perform fundamentally different operations:

- **Temperature** reshapes the probability distribution, changing how strongly the model favours high-probability tokens.
    
- **Top-P (nucleus sampling)** restricts the candidate pool to the highest-probability tokens whose cumulative probability reaches a specified threshold.
    

The key distinction is that **Temperature changes the relative probabilities, while Top-P removes candidates based on those probabilities.**

This makes it possible to encourage varied, less predictable output while limiting access to the least probable tokens.

## 1. Token Probability Distribution

During generation, an LLM calculates a probability distribution over its vocabulary for the next token.

Consider the incomplete sentence:

> The knight drew his...

An illustrative distribution might be:

|Token|Probability|
|---|---|
|sword|50%|
|blade|20%|
|weapon|10%|
|dagger|7%|
|breath|5%|
|horse|3%|
|Other tokens|5%|

These values are hypothetical and are used only to demonstrate sampling behaviour.

Without additional restrictions, every token with a nonzero probability is theoretically eligible for selection.

The model is much more likely to select `sword` than `horse`, but selection is probabilistic rather than necessarily deterministic.

## 2. Temperature — Reshaping Probabilities

Temperature changes the relative probabilities assigned to candidate tokens.

The usual mathematical transformation is:

[  
P_i = \frac{\exp(z_i/T)}{\sum_j \exp(z_j/T)}  
]

Where:

- (z_i) is the logit for token (i).
    
- (T) is the temperature.
    
- (P_i) is the resulting probability.
    

### Low Temperature

When temperature is below 1.0, the distribution becomes more concentrated around the highest-probability tokens.

For example:

|Token|Illustrative probability|
|---|---|
|sword|70%|
|blade|15%|
|weapon|6%|
|dagger|3%|
|Others|6%|

The model becomes more likely to select the statistically obvious continuation.

### High Temperature

When temperature exceeds 1.0, the distribution becomes flatter.

For example:

|Token|Illustrative probability|
|---|---|
|sword|30%|
|blade|20%|
|weapon|14%|
|dagger|10%|
|breath|8%|
|horse|5%|
|Others|13%|

The less probable candidates become more competitive.

### Important distinction

Temperature does not evaluate creativity, intelligence, coherence, or contextual suitability.

It only changes the relative probabilities.

Consequently:

- A creative continuation may become more likely.
    
- An unusual but appropriate continuation may become more likely.
    
- An incoherent continuation may also become more likely.
    

**Temperature increases sampling diversity; creativity is a possible consequence, not the mechanism itself.**

At temperature 1.0, the distribution is unchanged by temperature scaling. Temperature approaching zero increasingly favours the highest-probability token; exact zero handling depends on the sampler.

## 3. Top-P — Restricting the Candidate Pool

Top-P, also called nucleus sampling, restricts selection to a dynamically sized set of high-probability tokens.

The sampler:

1. Sorts candidate tokens by descending probability.
    
2. Adds candidates to the eligible pool in that order.
    
3. Stops when cumulative probability reaches or exceeds the Top-P threshold.
    
4. Excludes the remaining candidates.
    
5. Renormalizes the retained probabilities before sampling.
    

Consider the original distribution with **Top-P = 0.80**.

|Token|Probability|Cumulative|
|---|---|---|
|sword|50%|50%|
|blade|20%|70%|
|weapon|10%|80%|
|dagger|7%|87%|
|breath|5%|92%|

The eligible candidate pool becomes:

```
sword
blade
weapon
```

All remaining candidates are excluded.

The retained probabilities are then renormalized:

|Token|Original|After Top-P|
|---|---|---|
|sword|50%|62.5%|
|blade|20%|25.0%|
|weapon|10%|12.5%|

### Important distinction

**Top-P = 0.9 does not mean retaining 90% of the vocabulary.**

It means retaining the smallest group of highest-probability tokens whose cumulative probability reaches at least 90%.

The number of eligible tokens therefore changes dynamically with each generated token.

A strongly concentrated distribution may require only a few candidates, while a flatter distribution may require hundreds or thousands.

## 4. How Temperature and Top-P Interact

In the sampling pipeline discussed here, Temperature is applied before Top-P.

```
Model logits
     |
     v
Temperature scaling
     |
     v
Reshaped probability distribution
     |
     v
Top-P cumulative probability filtering
     |
     v
Renormalized eligible probabilities
     |
     v
Random token selection
```

This ordering is significant because Top-P operates on probabilities that Temperature has already changed.

### Low Temperature + Top-P 0.9

Suppose low Temperature produces:

|Token|Probability|Cumulative|
|---|---|---|
|sword|70%|70%|
|blade|15%|85%|
|weapon|6%|91%|

Top-P 0.9 retains three candidates.

### High Temperature + Top-P 0.9

Suppose higher Temperature produces:

|Token|Probability|Cumulative|
|---|---|---|
|sword|30%|30%|
|blade|20%|50%|
|weapon|14%|64%|
|dagger|10%|74%|
|breath|8%|82%|
|horse|5%|87%|
|shield|4%|91%|

The same Top-P value now retains seven candidates.

### Result

Increasing Temperature generally makes the distribution flatter, requiring more tokens to reach the same cumulative Top-P threshold.

Therefore:

**Higher Temperature can increase both the relative probability of less-obvious tokens and the number of tokens surviving Top-P filtering.**

The exact result depends on the original probability distribution.

## 5. Why Use Both Parameters?

It is possible to set Top-P to 1.0 and control variation entirely through Temperature.

However, this provides no nucleus-sampling cutoff.

### Temperature alone

Increasing Temperature makes unusual continuations more competitive, but also increases the probability of very unlikely candidates.

The sampler retains access to the entire distribution when Top-P is 1.0, assuming no other filtering.

### Temperature combined with Top-P

A higher Temperature can make less-obvious continuations competitive.

Top-P then excludes the lowest-probability tail of that reshaped distribution.

This provides two complementary controls:

|Parameter|Primary purpose|
|---|---|
|Temperature|Control the relative spread of probabilities|
|Top-P|Control how much cumulative probability mass remains eligible|

A useful mental model:

> **Temperature widens the possibilities. Top-P trims the least probable tail.**

This is particularly relevant for creative writing and roleplay, where predictable continuations can become repetitive but unrestricted sampling can undermine coherence.

## 6. The Probability Versus Sensibility Distinction

Neither parameter understands whether a token is sensible.

Top-P is not a quality filter.

It cannot distinguish between:

- A low-probability but contextually brilliant continuation.
    
- A low-probability nonsensical continuation.
    
- A high-probability but narratively boring continuation.
    
- A high-probability continuation that is inappropriate in the wider context.
    

All it considers is probability.

Consequently, a highly creative and appropriate token may be excluded simply because its probability falls below the cumulative cutoff.

Equally, an undesirable token can survive if its probability is sufficiently high.

This is a fundamental limitation of probability-based sampling.

## 7. Practical Implications for SillyTavern Roleplay

For roleplay, the objective is often to encourage varied and character-appropriate responses without introducing excessive unpredictability.

### Low Temperature + Low Top-P

- Strong preference for common continuations.
    
- Restricted candidate pool.
    
- Often predictable wording.
    
- Potentially repetitive responses.
    

### Low Temperature + High Top-P

- Larger candidate pool is permitted.
    
- High-probability tokens still dominate.
    
- Usually conservative despite the wider theoretical selection.
    

### High Temperature + Low Top-P

- Probabilities become more evenly distributed.
    
- Top-P restricts the available candidates.
    
- Increased variation may be partly or substantially cancelled by aggressive filtering.
    

### High Temperature + High Top-P

- More candidates become competitive.
    
- A wider range of continuations survives filtering.
    
- Greater potential for surprising prose.
    
- Increased risk of incoherence or inconsistent characterization.
    

### Counterintuitive behaviour

Consider two hypothetical configurations:

|Temperature|Top-P|
|---|---|
|1.3|0.50|
|0.9|0.95|

Despite its higher Temperature, the first configuration may produce less variation because Top-P aggressively restricts the candidate pool.

The second configuration may permit more varied output despite its lower Temperature.

This is not guaranteed for every probability distribution, but demonstrates why the parameters must be considered together.

## 8. Tuning Strategy

For practical experimentation:

1. Establish a baseline using the current model and prompt.
    
2. Keep Top-P fixed while adjusting Temperature.
    
3. Observe changes in variation, repetition, coherence, and characterization.
    
4. Keep Temperature fixed while adjusting Top-P.
    
5. Compare outputs using consistent prompts and, where supported, fixed seeds.
    
6. Change one parameter at a time before experimenting with combinations.
    

Do not assume a particular pair of values will behave identically across different models.

Model training, fine-tuning, quantization, prompt context, and additional sampling controls all influence the resulting distribution.

### Backend caveat

The precise sampling pipeline is implementation-dependent.

SillyTavern may expose sampling controls that are ultimately implemented by a backend such as Ollama, llama.cpp, or KoboldCpp.

Additional samplers, including Top-K, Min-P and repetition penalties, can modify the distribution or eligible candidate pool.

The order in which these operations are applied matters.

The Temperature → Top-P explanation describes the pipeline assumed in this discussion, not a guarantee that every backend applies samplers in that order.

## 9. Key Conclusions

1. **Temperature changes probability distribution shape.** Higher values flatten the distribution; lower values concentrate it.
    
2. **Top-P filters by cumulative probability mass.** It does not retain a fixed percentage or number of vocabulary tokens.
    
3. **The two parameters interact.** A flatter distribution generally means more tokens are required to reach the same Top-P threshold.
    
4. **Top-P is not a sensibility filter.** It removes low-probability candidates regardless of their contextual quality.
    
5. **Higher Temperature does not guarantee greater output diversity when Top-P is restrictive.**
    
6. **Both controls can be useful for roleplay.** Temperature encourages less predictable continuations, while Top-P limits access to the lowest-probability tail.
    
7. **Backend behaviour must be verified.** Other sampling controls and their execution order can change the outcome.
    

## Mental Model

The entire interaction can be reduced to two operations:

> **Temperature reshapes the probability curve.**
> 
> **Top-P cuts the tail off that reshaped curve.**

Temperature determines how evenly probability is distributed among possible next tokens.

Top-P determines which of those tokens remain eligible for selection.

Together, they control the balance between predictable and varied generation without making any direct judgement about the quality or sensibility of the resulting text.