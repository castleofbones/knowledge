---

type: reference  
status: draft  
created: 2026-10-18  
source: ""  
tags:

- sillytavern
    
- gemma-4
    
- ollama
    
- llm-sampling
    
- roleplay
    
- inference-configuration
    

---
## Overview

This article documents a recommended starting configuration for **Gemma 4 roleplay (RP) chat using SillyTavern and Ollama**.

The objective is to balance:

- Creative, varied dialogue and narration.
    
- Consistent character personality and behaviour.
    
- Narrative coherence and continuity.
    
- Reduced repetition and looping.
    
- Sufficient response length without unnecessarily verbose output.
    

The values are a **proposed tuning baseline**, not an experimentally verified optimum. Actual behaviour depends on the model variant, quantization, prompt format, backend and roleplay scenario.

## Recommended Sampling Configuration

|Setting|Recommended|Purpose|
|---|---|---|
|Max Response Tokens|1024|Limits response length|
|Temperature|1.2|Encourages variation and creativity|
|Top-P|0.95|Removes the low-probability tail|
|Top-K|64|Restricts candidate tokens|
|Min-P|0.02|Filters tokens unlikely relative to the best candidate|
|Repetition Penalty|1.05|Mildly discourages repeated tokens|
|Presence Penalty|0.0|Avoids penalising previously introduced concepts|
|Frequency Penalty|0.0|Avoids penalising recurring names and terminology|

These settings assume that the selected SillyTavern connection and inference backend support the corresponding parameters.

### Baseline Summary

```
Temperature:          1.20
Top-P:                0.95
Top-K:                64
Min-P:                0.02
Repetition Penalty:   1.05
Presence Penalty:     0.00
Frequency Penalty:    0.00
Max Response Tokens:  1024
```

## How the Sampling Parameters Interact

Language models generate text by repeatedly predicting a probability distribution over possible next tokens.

Sampling parameters modify this distribution or restrict which tokens can be selected.

### Temperature — 1.2

Temperature changes the relative differences between token probabilities.

- Below 1.0: Sharpens the distribution, favouring high-probability tokens.
    
- At 1.0: Leaves the probability distribution unchanged by temperature scaling.
    
- Above 1.0: Flattens the distribution, increasing the relative likelihood of less probable tokens.
    

For RP, a moderately elevated temperature can encourage less predictable dialogue, more varied descriptions and character responses.

However, excessive temperature can undermine consistency and coherence.

**Recommended starting point: 1.2.**

### Top-K — 64

Top-K restricts sampling to the K highest-probability tokens.

With Top-K set to 64, only the 64 most probable candidates remain eligible at that filtering stage.

This prevents extremely improbable tokens from entering the final sampling pool.

Top-K does not mean that 64 tokens are generated. It restricts the candidates for **each individual token decision**.

**Recommended starting point: 64.**

### Top-P — 0.95

Top-P, also called nucleus sampling, retains the smallest set of highest-probability tokens whose cumulative probability reaches or exceeds the configured threshold.

At 0.95, the sampling pool covers approximately 95% of the probability mass at the stage where Top-P is applied.

The number of eligible tokens therefore varies according to the distribution.

- A concentrated distribution may require only a few candidates.
    
- A flatter distribution may require many more.
    

Top-P complements temperature by excluding unlikely candidates while retaining a variable-sized pool.

**Recommended starting point: 0.95.**

### Min-P — 0.02

Min-P excludes tokens whose probabilities fall below a threshold relative to the most probable token.

For example, if the highest-probability token has probability 0.30, Min-P 0.02 gives a relative cutoff of:

```
0.30 × 0.02 = 0.006
```

Tokens with probabilities below 0.006 are excluded by this filter.

Unlike Top-P, Min-P evaluates probabilities relative to the strongest candidate rather than accumulating probability mass.

A small value provides relatively gentle filtering.

**Recommended starting point: 0.02.**

Min-P is an optional additional filter. It may be unnecessary when Top-K and Top-P already provide sufficient restriction.

### Repetition Penalty — 1.05

Repetition penalty discourages reuse of previously generated tokens.

Typical interpretation:

- 1.0: No repetition penalty.
    
- Above 1.0: Discourages repetition.
    
- Excessively high values: Can make natural language awkward.
    

Roleplay requires legitimate repetition of character names, locations, objects and recurring themes.

Consequently, strong repetition penalties can damage narrative continuity.

**Recommended starting point: 1.05.**

The effective behaviour also depends on the backend's repetition window and penalty implementation.

### Presence Penalty — 0.0

Presence penalty discourages tokens associated with material already present in the relevant context.

Unlike frequency penalty, its basic mechanism depends on whether a token has appeared rather than how often it appears.

This can encourage new vocabulary or concepts.

However, RP frequently revisits established characters, locations and plot elements.

A positive presence penalty may therefore work against narrative continuity.

**Recommended starting point: 0.0 (disabled).**

### Frequency Penalty — 0.0

Frequency penalty discourages tokens according to how frequently they have appeared in the relevant context.

This can reduce repetitive language but may also interfere with legitimate recurring terminology.

For RP, it is generally preferable to begin without frequency penalty and introduce it only when a specific repetition problem has been identified.

**Recommended starting point: 0.0 (disabled).**

### Max Response Tokens — 1024

This is the maximum number of output tokens permitted for a generation.

It is a hard output limit, not a creativity setting or a target response length.

The model can finish before reaching the limit.

For RP:

- Too low: Dialogue or narration may be truncated.
    
- Too high: Allows unnecessarily long responses if prompts do not control verbosity.
    
- Moderate: Provides room for substantial exchanges without encouraging unlimited output.
    

**Recommended starting point: 1024 tokens.**

The model's context window and maximum output allowance are separate constraints.

## Conceptual Sampling Pipeline

A simplified view of the process:

```
Model produces next-token logits
              |
              v
      Temperature scaling
              |
              v
     Probability distribution
              |
              v
        Top-K filtering
              |
              v
        Top-P filtering
              |
              v
        Min-P filtering
              |
              v
       Random token sample
              |
              v
     Append selected token
              |
              v
      Repeat until finished
```

This is a conceptual illustration, **not a guaranteed execution order**.

The actual ordering of sampling filters and penalties depends on the inference backend and its configuration. Repetition, presence and frequency penalties may be applied at different stages.

The important principle is that sampling decisions occur **for every generated token**, not once for the entire response.

## Recommended Tuning Procedure

Change one parameter at a time to make differences easier to attribute.

### Experiment 1 — Temperature

Keep all other settings fixed.

Test:

|Temperature|Expected Behaviour|
|---|---|
|1.0|More conservative baseline|
|1.2|Moderate creativity|
|1.4|Greater variation|
|1.5|More adventurous, potentially less consistent|

These are hypotheses to evaluate, not guaranteed outcomes.

Compare the same RP scenario across each setting.

Evaluate:

- Character consistency.
    
- Dialogue quality.
    
- Descriptive variation.
    
- Narrative coherence.
    
- Unwanted tangents.
    
- Repetition.
    

Use the same character card, scenario, context and generation settings wherever possible.

Because sampling is stochastic, compare multiple outputs per configuration rather than relying on a single generation.

### Experiment 2 — Min-P

After selecting a preferred temperature, compare:

```
Min-P: 0.00
Min-P: 0.02
```

Keep Top-K at 64 and Top-P at 0.95.

Determine whether Min-P meaningfully improves output quality or unnecessarily restricts variation.

If there is no observable benefit, leaving Min-P disabled simplifies the configuration.

### Experiment 3 — Repetition Penalty

Only adjust repetition penalty when a measurable repetition problem exists.

Compare:

```
Repetition Penalty: 1.00
Repetition Penalty: 1.05
Repetition Penalty: 1.10
```

Watch for:

- Repeated phrases.
    
- Recurring sentence structures.
    
- Repeated dialogue.
    
- Unnatural avoidance of character names.
    
- Deterioration in prose quality.
    

Do not automatically increase repetition penalty to address every form of repetition. Some repetition is caused by prompting, accumulated context or model behaviour rather than sampling configuration.

## Roleplay-Specific Considerations

### Character Consistency

Sampling settings cannot independently guarantee consistent character behaviour.

Character cards, system instructions, chat history, lorebooks and prompt construction also influence responses.

Higher temperature can improve variation but may increase departures from established personality or narrative constraints.

### Narrative Continuity

Presence and frequency penalties should remain disabled initially because roleplay naturally depends on recurring concepts.

Continuity should primarily be maintained through appropriate context management and prompting.

### Response Length

Maximum response tokens should be chosen according to the desired RP style.

A model that consistently produces overly long responses may require clearer narrative instructions rather than simply a lower token limit.

### Backend Compatibility

SillyTavern exposes sampling controls, but the inference backend ultimately determines which parameters are supported and how they are applied.

For Ollama, verify that the chosen connection mode actually transmits the intended options.

Do not assume that every visible SillyTavern control affects generation.

The exact Gemma 4 variant, quantization and Ollama version should be recorded when comparing experiments.

## Limitations and Verification Status

The recommended configuration is a practical starting hypothesis derived from sampling principles and the preceding discussion.

It has not been established here that:

- These values are optimal for every Gemma 4 variant.
    
- Every backend applies the filters in the same order.
    
- Ollama accepts every listed penalty through the selected SillyTavern connection.
    
- The settings have been benchmarked against a fixed RP test suite.
    

The earlier discussion referenced community recommendations around Temperature 1.0–1.5, Top-K 64 and Top-P 0.95. Those recommendations should be treated as external guidance rather than verified results from the local installation.

## References

- [SillyTavern Documentation](https://docs.sillytavern.app/)
    
- [Ollama Documentation](https://docs.ollama.com/)
    
- [Community LLM Settings Guide](https://huggingface.co/spaces/overhead520/LLM-Settings-Guide)
    

## Conclusion

For Gemma 4 roleplay in SillyTavern, begin with:

**Temperature 1.2, Top-P 0.95, Top-K 64, Min-P 0.02, Repetition Penalty 1.05, Presence Penalty 0 and Frequency Penalty 0.**

Use a 1024-token response limit initially.

Temperature is the first parameter worth experimenting with, followed by whether Min-P provides any measurable benefit.

Avoid simultaneously adjusting multiple sampling parameters. Establish a reproducible baseline and refine it using actual RP outputs rather than relying exclusively on theoretical behaviour.