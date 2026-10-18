---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- ai
    
- sillytavern
    
- ollama
    
- prompt-engineering
    
- prompt-construction
    
- presets
    
- context-management
    

---
## Purpose

This note explains how SillyTavern assembles the information presented to a language model, the role of presets in that process, and the implications for prompt quality, debugging and reproducibility.

The focus is on **prompt architecture and behaviour**, rather than individual generation parameters or backend implementation.

For the transport and response-processing stages, see [[SillyTavern Request and Response Pipeline]].

## 1. SillyTavern as a Prompt Assembly Engine

SillyTavern is more than a chat interface.

One of its principal responsibilities is assembling multiple sources of information into a model request.

Depending on the selected API mode and configuration, these sources may include:

- System and behavioural instructions
    
- Preset-defined prompt entries
    
- Character definitions and personality information
    
- Scenario and relationship information
    
- World Info / lorebook entries
    
- Author's Note and other injected instructions
    
- Conversation history
    
- Example dialogue
    
- User input
    
- Dynamically expanded macros
    

The model does not inherently understand where these elements originated.

It receives the resulting prompt or structured messages and generates a continuation based on that information.

**The quality of the assembled context can be as important as the model itself.**

## 2. Conceptual Prompt Construction

A simplified representation of the assembly process is:

```
Preset configuration
        │
        ├── System instructions
        ├── Prompt ordering
        ├── Injection positions
        └── Context formatting
                    │
Character data ─────┤
World Info ─────────┤
Author's Note ──────┤
Chat history ───────┤
User input ─────────┤
                    ▼
           Macro expansion
                    │
                    ▼
          Context budgeting
                    │
                    ▼
       Final prompt / messages
                    │
                    ▼
             Model request
```

This is a **conceptual model**, not a verified SillyTavern execution trace.

The real implementation may evaluate, insert, filter and transform these elements in different orders, depending on the API mode and enabled extensions.

Establishing the exact sequence is one of the objectives of the SillyTavern Prompt Workbench project.

## 3. The Role of Presets

A SillyTavern preset can influence substantially more than sampling settings.

Depending on the preset type and API mode, it may define or affect:

|Area|Potential influence|
|---|---|
|System instructions|Model behaviour and response expectations|
|Prompt entries|Which instructions are included|
|Prompt order|Relative position of instructions and context|
|Role assignment|Whether content is sent as system, user or assistant|
|Injection settings|Placement and depth of additional information|
|Context formatting|Treatment of text and message boundaries|
|Generation parameters|Sampling and output limits|
|Backend-specific options|Parameters passed to the selected API|

Not every preset exposes or controls every area.

It is important to distinguish between **prompt presets**, **generation presets** and other configuration layers when comparing behaviour.

A preset should not be treated as a single monolithic prompt.

## 4. Prompt Order and Instruction Priority

Prompt position can influence model behaviour.

Instructions may be affected by:

- Their role within the request
    
- Their position relative to other instructions
    
- Whether they conflict with more authoritative instructions
    
- Their proximity to the most recent user message
    
- The amount of intervening context
    

For chat-completion APIs, system, developer, user and assistant roles may have distinct semantics.

However, those semantics depend on the model and backend.

A local model may be trained to recognise particular role delimiters or chat templates, and its response to them may differ from that of a hosted model.

### Prompt order is not equivalent to instruction authority

Moving an instruction later in the prompt may increase its practical influence in some circumstances, but does not automatically give it greater formal priority.

Similarly, repeating an instruction does not guarantee stronger compliance.

Poorly organised presets can introduce conflicting or redundant directions that reduce output quality.

## 5. Character Cards and Contextual Information

Character information provides the model with the fictional context required for roleplay.

Typical elements include:

- Character description
    
- Personality
    
- Scenario
    
- Example dialogue
    
- Character-specific instructions
    

The effective placement of this information depends on the selected prompt configuration.

### Example dialogue

Example dialogue can be particularly influential because it demonstrates the desired response style rather than merely describing it.

However, examples may also encourage repeated phrasing or overly rigid response structures.

### World Info

World Info entries introduce additional context, often based on activation rules.

Their presence in the final request may depend on:

- Keyword matching
    
- Activation conditions
    
- Available context budget
    
- Insertion configuration
    
- Extension behaviour
    

A World Info entry existing in the database does not necessarily mean it was included in a particular generation.

This distinction is important when diagnosing apparently forgotten information.

## 6. Macros and Dynamic Content

SillyTavern supports macros that substitute dynamic values into prompt text.

Conceptually, a prompt entry might contain:

```
You are speaking with {{user}}.
```

The model should receive the expanded value rather than the unresolved placeholder, assuming the macro is supported and processed in that context.

Macros introduce another source of variability.

Two requests using the same preset file may differ because:

- Character or user information changed
    
- Conversation state changed
    
- Dynamic values changed
    
- Extension-provided information changed
    

For reproducible comparisons, it is therefore insufficient to record only the preset filename.

The **expanded prompt actually sent to the model** is the more useful evidence.

## 7. Context Budgeting and Truncation

Every model request operates within a context budget.

SillyTavern must account for prompt content, conversation history and any reserved generation allowance according to the selected backend and configuration.

When available context is insufficient, some content may be omitted or truncated.

The exact behaviour depends on the prompt configuration and implementation.

### Why this matters

A character may appear to forget information because the relevant message was no longer included in the request.

A lorebook entry may fail to influence generation because it was not activated or did not survive context budgeting.

An instruction may appear ineffective because it was removed, displaced or contradicted elsewhere.

These are prompt-construction issues, not necessarily model-memory failures.

### Debugging principle

When investigating missing context, determine:

1. Whether the information existed in the source data.
    
2. Whether it was eligible for inclusion.
    
3. Whether it was inserted into the assembled prompt.
    
4. Whether it survived context budgeting.
    
5. Whether it reached the inference backend.
    

Only then evaluate whether the model failed to use the information.

## 8. Chat Completion versus Text Completion

SillyTavern supports different approaches to formatting requests.

### Chat Completion

The request typically contains structured messages with roles and content.

Conceptually:

```
{
  "messages": [
    {
      "role": "system",
      "content": "Behavioural instructions"
    },
    {
      "role": "user",
      "content": "Hello"
    }
  ]
}
```

The backend and model-specific chat template determine how those messages are ultimately represented to the model.

### Text Completion

The frontend or backend constructs a textual prompt, often using instruct formatting and role delimiters.

Conceptually:

```
<system>
Behavioural instructions
</system>

<user>
Hello
</user>

<assistant>
```

This is illustrative formatting, not a literal SillyTavern template.

### Practical consequence

The same character and preset instructions can behave differently between chat-completion and text-completion configurations.

Role representation, delimiters, stop sequences and model chat-template expectations all matter.

Our local Gemma 4 configuration uses SillyTavern's Text Completion mode with Ollama.

That configuration should be treated as part of the experimental baseline when comparing presets.

## 9. Case Study — Pura Director 15.1 versus 16.0

Our investigation identified a meaningful difference in roleplay quality between two versions of the Pura Director preset.

|Configuration|Observed behaviour|
|---|---|
|Pura Director 15.1|Produced satisfactory roleplay results|
|Pura Director 16.0|Produced noticeably poorer results|

The degradation was observed using both local Ollama and OpenRouter with Gemma 4.

### What this suggests

Because the issue appeared across two inference services, it was reasonable to investigate the preset itself rather than assuming a local Ollama fault.

However, this does not establish which specific change caused the regression.

Potential contributors include:

- Changed behavioural instructions
    
- Different prompt ordering
    
- Additional or removed prompt entries
    
- Conflicting instructions
    
- Changed formatting or injection configuration
    

These remain hypotheses until the effective requests are compared.

### Recommended comparison

Use the same:

- Character and scenario
    
- Conversation history
    
- Model and model version
    
- Generation parameters
    
- Context configuration
    
- User message
    

Then capture and compare the actual requests generated under each preset.

This allows investigation of the **effective prompt differences**, rather than merely comparing the preset files.

### Source

Pura preset repository:

[https://github.com/platberlitz/platberlitz.github.io/tree/main/preset](https://github.com/platberlitz/platberlitz.github.io/tree/main/preset)

## 10. Formatting Is Part of the Pipeline

An important lesson from our SillyTavern troubleshooting was that displayed output can differ from the model's raw response.

We investigated disappearing paragraph spacing and identified **Collapse Consecutive Newlines** under:

`Advanced Formatting → Context Formatting`

This setting is relevant to formatting behaviour, but it is important to distinguish prompt-side formatting from response-side processing.

A definitive explanation of where a transformation occurs requires inspection of the relevant code path and captured input/output.

The broader principle is:

> [!important]  
> Never assume that a formatting problem visible in the chat interface originated in the model.

Inspect the raw response, backend handling and frontend rendering before assigning responsibility.

See [[SillyTavern Request and Response Pipeline]].

## 11. Reproducibility and Prompt Inspection

A reproducible preset comparison requires more than exporting two preset files.

Ideally, capture:

|Information|Purpose|
|---|---|
|SillyTavern version|Identify the assembly implementation|
|Preset identifier/version|Establish configuration|
|Model and backend|Control inference differences|
|Ordered messages|Reveal effective prompt structure|
|Expanded macros|Show actual submitted content|
|Role assignments|Identify instruction placement|
|Context usage|Reveal budgeting and omissions|
|Generation parameters|Control sampling differences|
|Raw request JSON|Preserve the actual API request|
|Raw response|Separate model output from presentation|

These records form the basis of reliable debugging and regression testing.

## 12. SillyTavern Prompt Workbench

The proposed SillyTavern Prompt Workbench is intended to inspect the actual prompt assembly process.

Its central architectural principle is:

**Capture what SillyTavern constructs rather than independently recreating its prompt assembly logic.**

Reimplementing the assembly process would introduce a second interpretation of SillyTavern's behaviour, creating opportunities for the debugger itself to be wrong.

### Intended capabilities

- Capture final outbound requests
    
- Inspect ordered prompt messages
    
- Compare preset versions
    
- Identify changes in instruction placement
    
- Record generation parameters
    
- Inspect context budgeting
    
- Trace prompt content to its source where instrumentation permits
    
- Support reproducible regression comparisons
    

### Implementation distinction

Capturing the final request proves what was sent.

Determining **why** a particular message was included, transformed or omitted requires deeper instrumentation of the assembly pipeline.

These are separate levels of observability and should be implemented incrementally.

The project-specific implementation plan remains in the SillyTavern Prompt Workbench project documentation rather than this general reference note.

## 13. Practical Troubleshooting Approach

When a preset behaves unexpectedly:

1. Reproduce the behaviour with a known conversation.
    
2. Record the model, backend and generation settings.
    
3. Inspect the assembled prompt where possible.
    
4. Look for conflicting or duplicated instructions.
    
5. Check role assignment and prompt order.
    
6. Verify World Info and injected content.
    
7. Check context budgeting and truncation.
    
8. Compare the effective request with a known-good preset.
    
9. Inspect raw model output before evaluating displayed formatting.
    
10. Change one relevant configuration element at a time.
    

This avoids attempting to solve prompt-construction problems by repeatedly adjusting sampling parameters.

## Related Notes

- [[LLM Generation Parameters — A Practical Guide]]
    
- [[Quantisation, VRAM and Context]]
    
- [[SillyTavern Request and Response Pipeline]]
    
- [[SillyTavern Docker Patching and Maintenance]]
    
- [[Local Coding Agents]]
    

## Key Takeaway

**A SillyTavern preset is not merely a collection of generation settings. It participates in defining the instructions, structure and context presented to the model.**

When two presets produce different results, the most useful evidence is the actual assembled request, not just the differences between their configuration files.

Understanding that distinction is fundamental to reliable prompt debugging and the design of the SillyTavern Prompt Workbench.