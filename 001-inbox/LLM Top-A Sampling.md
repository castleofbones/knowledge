---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- llm
    
- token-sampling
    
- top-a
    
- sillytavern
    
- text-generation
    

---

# Top-A Sampling in Large Language Models

## Overview

**Top-A is an adaptive token-sampling filter** that removes low-probability tokens according to the probability of the model's most likely next token.

Unlike Top-K, which retains a fixed number of candidates, or Top-P, which retains a cumulative probability mass, Top-A adjusts its filtering threshold according to the model's confidence in its leading prediction.

The underlying principle is:

> The more confident the model is about its best token, the more aggressively Top-A filters unlikely alternatives.

When the model is uncertain and the probability distribution is relatively flat, Top-A permits a wider selection of candidate tokens.

This makes Top-A potentially useful for creative text generation and roleplay, where predictable grammatical continuations and genuinely open-ended decisions require different degrees of flexibility.

## How Top-A Works

Top-A calculates a minimum probability threshold using:
$$P_{\text{cutoff}} = A \times P_{\text{max}}^2  $$
Where:

- ($A$) is the configured Top-A value.
    
- ($P_{\text{max}}$) is the probability of the most likely token.
    
- ($P_{\text{cutoff}}$) is the minimum probability required for a token to remain eligible.
    

Any token whose probability falls below this threshold is excluded from sampling.

The remaining probabilities are renormalized before token selection.

### Example 1: High Model Confidence

Assume Top-A is configured to `0.2`.

The model produces the following probability distribution:

|Token|Probability|
|---|---|
|the|50%|
|a|18%|
|his|10%|
|that|7%|
|purple|4%|
|llama|1%|
|wibble|0.1%|

The most probable token is `the`, with a probability of 50%.

The cutoff becomes:
$$0.2 \times 0.50^2 = 0.05  $$
Therefore, tokens must have at least a 5% probability to remain eligible.

|Token|Probability|Result|
|---|---|---|
|the|50%|Retained|
|a|18%|Retained|
|his|10%|Retained|
|that|7%|Retained|
|purple|4%|Removed|
|llama|1%|Removed|
|wibble|0.1%|Removed|

The model retains four candidates.

**Interpretation:** When one continuation is substantially more likely than the others, Top-A eliminates relatively weak alternatives.

### Example 2: Low Model Confidence

Consider a more ambiguous continuation:

|Token|Probability|
|---|---|
|walked|12%|
|looked|11%|
|turned|10%|
|stepped|9%|
|glanced|8%|
|smiled|7%|

The leading probability is now only 12%.

Using the same Top-A value:
$$0.2 \times 0.12^2 = 0.00288$$  The cutoff is now just **0.288%**.

Every token in the example survives, along with potentially many additional candidates.

**Interpretation:** When the model is less confident about the next token, Top-A permits substantially greater variation.

This is particularly relevant when generating dialogue, character actions, descriptions, and narrative developments.

## Comparison With Other Sampling Filters

|Sampler|Selection mechanism|Adaptive behavior|
|---|---|---|
|Top-K|Retains the K highest-probability tokens|Fixed candidate count|
|Top-P|Retains the smallest leading set reaching cumulative probability P|Candidate count varies|
|Min-P|Retains tokens above a fraction of the highest probability|Linear relationship to maximum probability|
|Top-A|Retains tokens above A multiplied by the squared maximum probability|Quadratic relationship to maximum probability|

### Top-A vs Min-P

Both filters use the probability of the leading token, but their calculations differ.

Min-P:
$$P_{\text{cutoff}} = M \times P_{\text{max}}$$  
Top-A:  
$$P_{\text{cutoff}} = A \times P_{\text{max}}^2$$  
For illustration, assume both settings are `0.2`.

| Highest token probability | Min-P cutoff | Top-A cutoff |
| ------------------------- | ------------ | ------------ |
| 80%                       | 16%          | 12.8%        |
| 50%                       | 10%          | 5%           |
| 25%                       | 5%           | 1.25%        |
| 12%                       | 2.4%         | 0.288%       |
| 5%                        | 1%           | 0.05%        |

Because probabilities lie between zero and one, squaring the maximum probability produces a smaller number.

Therefore, **at equal numerical settings, Top-A is generally less restrictive than Min-P**, especially when the leading probability is small.

This is an important distinction: Top-A's threshold changes quadratically with confidence, but it does not necessarily filter more aggressively than Min-P.

The two settings should not be treated as interchangeable simply because they accept similar numerical values.

## Interaction With Temperature

Temperature changes the model's token probability distribution before sampling filters are applied, assuming the backend uses that processing order.

Higher temperature generally flattens the distribution:

- The leading token becomes less dominant.
    
- Alternative tokens receive greater relative probability.
    
- Top-A's calculated threshold decreases as the maximum probability falls.
    

Lower temperature generally sharpens the distribution:

- The leading token becomes more dominant.
    
- Less likely alternatives receive lower relative probability.
    
- Top-A's threshold increases as the maximum probability rises.
    

Consequently, temperature and Top-A can reinforce one another's effects on the available candidate pool.

However, the precise outcome depends on the backend's sampler implementation and processing order.

## Using Top-A for Roleplay

Roleplay generation involves different types of decisions.

Some continuations are relatively predictable:

- Completing grammatical structures.
    
- Following established dialogue patterns.
    
- Continuing familiar descriptions.
    

Others are more open-ended:

- Choosing a character's next action.
    
- Developing emotional reactions.
    
- Introducing narrative complications