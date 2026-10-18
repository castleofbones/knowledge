---
type: reference
status: active
created: 2026-10-18
source:
tags:
  - llm
  - generation-settings
  - llm-sampling
  - inference-settings
  - inference-configuration
  - sillytavern
  - ollama
---
## Purpose

This note brings together the generation parameters used by local LLMs, particularly when running models through Ollama and SillyTavern.

It focuses on **how the parameters interact, how to approach tuning, and how to recognise problems that sampling adjustments cannot solve**.

The detailed behaviour of individual parameters is covered in their dedicated notes.

## Parameter Reference

| Parameter              | Primary function                                       |
| ---------------------- | ------------------------------------------------------ |
| [[Max Tokens]]         | Limits the number of tokens generated                  |
| [[Temperature]]        | Adjusts the probability distribution                   |
| [[Top-P]]              | Restricts candidates by cumulative probability         |
| [[Top-K]]              | Restricts candidates by ranking                        |
| [[Min-P]]              | Filters candidates relative to the highest probability |
| [[Top-A]]              | Applies a probability-dependent candidate threshold    |
| [[Repetition Penalty]] | Discourages reuse of previously generated tokens       |
| [[Presence Penalty]]   | Discourages tokens already present in the context      |
| [[Frequency Penalty]]  | Penalises tokens according to their occurrence count   |

Not every inference engine supports every parameter, and implementations can differ.

## How the Parameters Work Together

Generation settings are best understood as several interacting groups.

### Probability Distribution

**Temperature** influences the relative likelihood of candidate tokens.

Higher temperatures make alternatives more competitive; lower temperatures favour the strongest candidates.

### Candidate Filtering

**Top-P, Top-K, Min-P and Top-A** restrict which candidates remain eligible for selection.

These filters operate differently, and combining them can produce unexpectedly restrictive sampling.

For example, increasing temperature may broaden the probability distribution, but aggressive filtering can subsequently remove many of those alternatives.

### Repetition Control

**Repetition, presence and frequency penalties** influence the likelihood of previously encountered tokens.

They can help with repetitive output, but excessive penalties may interfere with names, terminology, grammatical consistency and recurring narrative elements.

### Output Limits

**Maximum output tokens** determines when generation must stop if no earlier stopping condition occurs.

It does not instruct the model to produce a particular response length.

### Implementation Matters

The conceptual relationship is:

```
Model predictions
       ↓
Probability adjustments
       ↓
Candidate filtering
       ↓
Token selection
       ↓
Repeat until stopping condition
```

This is not a universal processing order. Actual sampler ordering and supported features depend on the inference engine.

Consequently, the same parameter values may behave differently across Ollama, llama.cpp, KoboldCpp and other backends.

## Establishing a Baseline

Before tuning a model, record the environment:

- Model identifier and quantisation
    
- Inference engine and version
    
- Prompt or SillyTavern preset version
    
- Context window and output allowance
    
- Reasoning/thinking configuration
    
- Sampling parameters
    
- Seed, where supported
    

Without a stable baseline, it becomes difficult to attribute changes in output quality to individual settings.

For roleplay, compare multiple responses to the same scenario rather than judging a configuration from a single generation.

## Practical Tuning Strategy

The most useful approach is to change one parameter, or one closely related group, at a time.

|Observed behaviour|First investigation|
|---|---|
|Responses are too predictable|Temperature and candidate filtering|
|Responses become incoherent|Temperature, filtering and prompt quality|
|Repeated phrases or vocabulary|Repetition penalties and recent context|
|Repetitive narrative patterns|Prompt structure, preset instructions and model behaviour|
|Responses end abruptly|Maximum output tokens and stop conditions|
|Characters lose consistency|Context construction and conflicting instructions|
|Responses are unusually short|Prompt instructions, stopping behaviour and output limits|
|Structured output is malformed|Schema constraints, prompt structure and sampling variability|

A key distinction is that **not every generation problem is a sampling problem**.

Changing temperature will not reliably correct a badly constructed prompt. Increasing repetition penalties will not necessarily fix a preset that repeatedly encourages the same narrative behaviour.

## Practical Roleplay Baseline

The following is a starting point for experimentation, not a universal optimum.

|Parameter|Initial value|
|---|---|
|Temperature|0.8|
|Top-P|0.9|
|Top-K|40|
|Min-P|0.05|
|Top-A|Disabled|
|Repetition penalty|1.05|
|Presence penalty|0|
|Frequency penalty|0|
|Maximum output tokens|1,024–2,048|

Prefer model-author recommendations where available.

Fine-tuned, abliterated and quantised variants may behave differently from their base models.

Avoid assuming that every parameter displayed in SillyTavern is necessarily supported or honoured by the selected backend.

## Lessons from SillyTavern and Ollama

Our investigations have highlighted several practical considerations.

**Preset quality matters.** A change to prompt instructions can have a greater effect on roleplay quality than adjustments to sampling settings.

**Reasoning configuration matters.** Thinking and non-thinking modes may require different request parameters, and incorrect handling can result in missing or unexpected output.

**Frontend behaviour matters.** What appears in the chat interface is not necessarily identical to the raw model response.

**Backend compatibility matters.** A setting visible in SillyTavern may be unsupported, ignored or translated differently by the inference backend.

These are reasons to inspect the actual requests and responses before attributing every problem to the model or its generation parameters.

See [[SillyTavern Prompt Construction and Presets]] and [[SillyTavern Request and Response Pipeline]].

## Recommended Experiment Workflow

1. Establish a known-good model, preset and sampler configuration.
    
2. Save the baseline before making changes.
    
3. Identify one specific behaviour to improve.
    
4. Determine whether the likely cause is sampling, prompting, context or response processing.
    
5. Change the smallest relevant set of settings.
    
6. Compare several generations using equivalent prompts.
    
7. Keep improvements and revert unsuccessful experiments.
    
8. Record the result for future reference.
    

For preset comparisons, preserve the sampling settings so that differences in output can be attributed more confidently to the prompt configuration.

## Related Notes

### Individual parameters

- [[Max Tokens]]
    
- [[Temperature]]
    
- [[Top-P]]
    
- [[Top-K]]
    
- [[Min-P]]
    
- [[Top-A]]
    
- [[Repetition Penalty]]
    
- [[Presence Penalty]]
    
- [[Frequency Penalty]]
    

### Broader topics

- [[Quantisation VRAM and Context]]
    
- [[SillyTavern Prompt Construction and Presets]]
    
- [[SillyTavern Request and Response Pipeline]]
    
- [[Local Coding Agents]]
    

## Key Takeaway

**Generation parameters influence how a model chooses tokens. They do not compensate reliably for poor prompts, missing context, incompatible backend settings or frontend processing problems.**

Understand the individual parameters, establish a reproducible baseline, and tune only what the observed behaviour justifies.