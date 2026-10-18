---
type: reference
status: active
created: 2026-10-18
source: ""
tags:
  - llm
  - token-generation
  - inference-settings
  - sillytavern
  - ollama
  - max-tokens
---
## Overview

The **Max Tokens** generation parameter defines the maximum number of output tokens an LLM may generate during a single response.

It does **not** instruct the model to produce a response of that length, nor does it directly control verbosity.

An LLM can generate a short response and stop naturally, even when its configured token allowance is considerably larger.

Understanding the distinction between **natural stopping** and **forced termination** is particularly important when configuring conversational models for roleplay (RP) in SillyTavern.

## How an LLM Decides When to Stop

An autoregressive LLM generates output by repeatedly predicting the next token based on the preceding context.

Depending on the model and inference system, generation may end through several mechanisms:

1. **End-of-sequence (EOS) token:** The model generates a special token indicating that its response is complete.
    
2. **End-of-turn token:** Some chat models use a dedicated token to indicate completion of the assistant's turn.
    
3. **Stop sequences:** The inference engine detects a configured sequence and terminates generation.
    
4. **Maximum token limit:** The inference engine terminates generation after the permitted output token count is exhausted.
    

The first two mechanisms are learned behaviours associated with the model's training and conversation formatting.

The final mechanism is an externally enforced limit.

### Natural Stopping

Consider a model configured with:

```
max_tokens: 512
```

The model might produce:

> Yes, I agree. The second option seems better.

If it subsequently generates its appropriate end-of-turn token, generation stops.

The model does not need to consume its entire 512-token allowance.

### Forced Termination

Alternatively, a verbose model might produce:

> Yes, I agree. The second option seems better. There are several reasons for this. Firstly...

If the model continues generating until it reaches 512 tokens, the inference engine terminates the response.

This can result in incomplete sentences, interrupted dialogue or unfinished narrative descriptions.

**Max Tokens is therefore a hard ceiling, not a target response length.**

## Does the Model Know Its Token Budget?

Not necessarily.

In typical inference configurations, `max_tokens` is supplied to the inference engine as a generation parameter rather than as an explicit instruction within the model's prompt.

The model generally does not receive a message equivalent to:

> You have 512 tokens available. Adjust your response accordingly.

Instead, it continues predicting tokens until it produces an appropriate stopping token or an external stopping condition is reached.

Some systems explicitly communicate a response budget through prompting or other mechanisms. However, this behaviour should not be assumed merely because a maximum output length has been configured.

## What Actually Controls Verbosity?

Response length emerges from several interacting influences.

### 1. Model Training

Different models learn different conversational patterns.

Some favour concise answers, while others tend toward extended explanations, narration or elaborate descriptions.

These tendencies can exist even when models use identical inference settings.

### 2. System Prompts and Instructions

Explicit instructions can influence the expected length and style of responses.

For example:

```
Use concise, natural conversational responses.
Avoid unnecessary elaboration.
Expand descriptions only when the scene requires it.
```

For RP, instructions should generally encourage context-appropriate responses rather than enforce a rigid length for every turn.

### 3. Character Cards and Examples

In SillyTavern, character definitions and example dialogue can significantly influence response length.

A character card containing consistently long, descriptive messages may encourage the model to reproduce that pattern.

Conversely, examples of concise, natural dialogue can reinforce shorter conversational exchanges.

### 4. Conversation History

The model conditions its output on the existing conversation.

If previous assistant messages contain several paragraphs of narration, the model may continue that style.

This can create a feedback loop in which verbose responses encourage further verbosity.

### 5. Sampling Parameters

Parameters such as temperature, Top-P, Top-K and repetition penalties affect token selection.

They may indirectly influence response length and stopping behaviour, but they are not dedicated verbosity controls.

Changing sampling parameters solely to shorten responses can introduce unintended changes to coherence, creativity or conversational style.

## Practical Configuration for SillyTavern RP

For natural conversational RP, the objective is usually to allow responses of varying lengths without encouraging unnecessary elaboration.

An illustrative target might be:

|Parameter|Example|
|---|---|
|Typical conversational response|50–200 tokens|
|Occasional extended response|200–400 tokens|
|Maximum output allowance|512 tokens|

These values are examples rather than universally optimal settings.

The appropriate allowance depends on the model, character, prompt format and desired RP style.

A 512-token ceiling provides room for occasional longer responses while limiting excessive generation.

However, it does not guarantee that the model will normally respond within 50–200 tokens.

### Why Simply Reducing Max Tokens Is Not Ideal

Suppose a character regularly produces unnecessarily long responses.

Reducing the maximum from 512 to 128 tokens may appear to solve the problem.

However, this only restricts the available output.

It does not change the model's underlying tendency toward verbosity.

Consequently, responses may become:

- Abruptly truncated.
    
- Incomplete in the middle of dialogue.
    
- Missing important narrative information.
    
- Unable to accommodate scenes that genuinely require longer responses.
    

A better approach is to address verbosity through prompt instructions, character examples and conversation formatting, using Max Tokens as a safety limit.

## Diagnosing Unexpectedly Long Responses

When an RP model consistently generates excessive text, investigate the following:

|Area|What to examine|
|---|---|
|Model behaviour|Does the model naturally favour long responses?|
|System prompt|Are concise responses encouraged?|
|Character card|Does the description encourage excessive narration?|
|Example dialogue|Are all examples unusually long?|
|Conversation history|Is previous verbosity reinforcing itself?|
|Prompt formatting|Are assistant-turn boundaries correctly represented?|
|Stop sequences|Does the inference backend recognise the model's stopping tokens?|
|Sampling configuration|Are settings affecting coherence or stopping behaviour?|

A useful diagnostic is to compare responses to identical prompts while varying only one configuration element at a time.

This helps distinguish prompt-driven verbosity from model tendencies and inference configuration problems.

## Key Distinction

**Natural stopping** is model-driven:

> The response is complete, so generation ends.

**Maximum token termination** is inference-engine-driven:

> The permitted output length has been reached, so generation stops.

These mechanisms can produce dramatically different results even when the same maximum token allowance is configured.

## Conclusion

Max Tokens should be understood primarily as an **output safety limit**, not a verbosity setting.

For SillyTavern RP:

- Give the model enough output allowance to handle naturally longer responses.
    
- Encourage appropriate response length through prompting and examples.
    
- Avoid using aggressive token limits as a substitute for correcting verbose behaviour.
    
- Investigate model-specific stopping behaviour when responses are repeatedly truncated or fail to terminate naturally.
    

The practical principle is simple:

**Max Tokens determines how long the model is allowed to speak, not how long it should speak.**