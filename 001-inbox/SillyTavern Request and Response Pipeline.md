---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- ai
    
- sillytavern
    
- ollama
    
- request-pipeline
    
- response-processing
    
- debugging
    
- reasoning
    

purpose: knowledge
---
## Purpose

This note describes the request and response lifecycle between SillyTavern and a local inference backend, using Ollama as the primary example.

Its purpose is to establish **where information can be transformed, omitted or misinterpreted**, and how to isolate problems without immediately blaming the model, preset or frontend.

Prompt assembly and preset behaviour are covered in [[SillyTavern Prompt Construction and Presets]].

Container deployment, persistent patches and upgrade procedures belong in [[SillyTavern Docker Patching and Maintenance]].

## 1. Pipeline Overview

The conceptual request and response path is:

```
SillyTavern Frontend
        │
        │ Prompt configuration
        │ Generation settings
        │ Assembled context
        ▼
SillyTavern Backend
        │
        │ API request construction
        │ Parameter translation
        │ Authentication / headers
        ▼
Inference Backend
        │
        │ Model execution
        │ Token generation
        ▼
SillyTavern Backend
        │
        │ Response parsing
        │ Streaming handling
        │ Content extraction
        ▼
SillyTavern Frontend
        │
        │ Post-processing
        │ Formatting
        │ Rendering
        ▼
Displayed Chat Message
```

This is an architectural overview rather than a verified execution trace of every SillyTavern code path.

The precise implementation depends on the SillyTavern version, selected API mode and backend.

## 2. Request Construction

Once SillyTavern has assembled the prompt, it must construct a request compatible with the selected inference API.

That request can contain:

- Model identifier
    
- Prompt text or structured messages
    
- Sampling parameters
    
- Context and output limits
    
- Stop sequences
    
- Streaming configuration
    
- Backend-specific options
    

The distinction between a frontend setting and an actual outbound API parameter is important.

A value displayed in SillyTavern is not proof that the backend received it.

### Three separate questions

For any parameter, establish:

1. **Configured:** Is the parameter set in SillyTavern?
    
2. **Transmitted:** Is it present in the outbound request?
    
3. **Applied:** Does the inference backend recognise and honour it?
    

These are separate conditions.

A parameter may be configured but omitted, transmitted but ignored, or transformed into a backend-specific equivalent.

## 3. API Modes and Request Formats

SillyTavern can communicate with inference engines through different API modes.

Two important categories are Text Completion and Chat Completion.

### Text Completion

The request generally centres on a textual prompt or backend-specific completion structure.

The frontend or backend may be responsible for applying instruct formatting and role delimiters.

### Chat Completion

The request generally uses structured messages with roles such as `system`, `user` and `assistant`.

The inference backend may then apply a model-specific chat template.

### Why this matters

The same model can behave differently when the request format changes.

Differences may include:

- Role interpretation
    
- Chat-template application
    
- Stop-token behaviour
    
- Reasoning controls
    
- Parameter availability
    
- Response parsing
    

For debugging, always record the actual SillyTavern API mode.

Do not assume that two configurations are equivalent merely because they use the same model.

## 4. Local Ollama Configuration

Our established local arrangement is:

|Component|Location|
|---|---|
|SillyTavern|Raspberry Pi 5, Docker|
|Inference engine|Ollama on Windows 11|
|GPU|NVIDIA RTX 5070 Ti, 16 GB|
|Local model|Gemma 4 abliterated 12B QAT|
|Network|Local LAN|
|SillyTavern API mode|Text Completion|

The Ollama host has been configured to accept connections from other devices on the LAN.

A previous `ECONNREFUSED` problem was resolved by correcting the Ollama listening configuration and host connectivity.

### Connectivity checks

From a machine with access to the Ollama service:

```
curl http://10.10.0.83:11434/api/tags
```

Or, where `curl` is unavailable:

```
wget -qO- http://10.10.0.83:11434/api/tags
```

A successful response establishes basic API connectivity.

It does **not** establish that SillyTavern is constructing the correct request or that a model can generate successfully.

The IP address is specific to the documented local environment and should be updated if the network configuration changes.

## 5. Reasoning and Thinking Controls

Some models expose reasoning or thinking behaviour through backend-specific parameters.

These controls are not necessarily interchangeable across inference engines.

For example, one backend may accept a reasoning-effort value while another uses a boolean thinking option or a model-specific template setting.

### Our Gemma 4 investigation

We modified SillyTavern to support forwarding a `reasoning_effort` value of `none` for our Ollama configuration.

The work involved two areas:

- Frontend request construction in `public/scripts/openai.js`
    
- Backend handling in `src/endpoints/backends/chat-completions.js`
    

The objective was to ensure that the intended reasoning configuration reached the inference backend rather than remaining solely a frontend setting.

### Important distinction

The existence of a `reasoning_effort` field in an intermediate request does not automatically prove that the inference backend interpreted it correctly.

Verification requires examining the final outbound request and confirming the backend's supported API contract.

### Lessons learned

When a model produces empty, truncated or unexpected responses, consider:

- Whether reasoning is enabled or disabled
    
- Whether the backend recognises the supplied reasoning parameter
    
- Whether generated content is being returned in a reasoning field rather than a normal content field
    
- Whether the response parser handles the returned structure
    
- Whether stopping conditions terminate generation unexpectedly
    

These are distinct from ordinary sampling problems.

## 6. Streaming Versus Non-Streaming Responses

Inference APIs may return a complete response or stream incremental output.

### Non-streaming

The backend returns a completed response object.

This is often easier to inspect because the complete response structure is available at once.

### Streaming

The backend sends a sequence of partial results.

SillyTavern must interpret those results, extract relevant content and assemble the displayed response.

Potential complications include:

- Partial tokens or content fragments
    
- Separate reasoning and content fields
    
- Stream termination
    
- Stop conditions
    
- Unexpected response structures
    
- Errors occurring after generation has begun
    

A response may therefore fail during streaming even if the model itself generated valid output.

### Debugging approach

Where supported, compare streaming and non-streaming behaviour using equivalent requests.

If non-streaming works but streaming fails, investigate stream parsing and response assembly before changing the prompt or model.

## 7. Response Extraction

The inference backend may return more information than the text intended for display.

Depending on the API, a response can include:

- Generated content
    
- Reasoning or thinking content
    
- Finish reason
    
- Token usage
    
- Model metadata
    
- Tool calls
    
- Error information
    

SillyTavern must identify the appropriate fields and convert them into the representation expected by the frontend.

### Empty response does not always mean empty generation

An apparently blank message may result from:

1. The model generating no visible content.
    
2. The backend returning content in an unexpected field.
    
3. A response parser discarding content.
    
4. A stopping condition ending generation.
    
5. Frontend processing removing or hiding content.
    

The correct diagnosis depends on identifying the first stage where the expected text disappears.

## 8. Frontend Post-Processing and Rendering

After receiving model output, SillyTavern may apply formatting or other transformations before displaying the message.

This can create a difference between:

- Raw model output
    
- Backend-processed output
    
- Frontend message content
    
- Rendered chat display
    

These should be treated as separate representations when debugging.

### Our newline investigation

We previously encountered a problem where paragraph spacing disappeared in SillyTavern.

The relevant setting was identified as:

**Advanced Formatting → Context Formatting → Collapse Consecutive Newlines**

This demonstrated the importance of checking SillyTavern's formatting configuration before assuming the model generated incorrectly formatted text.

However, the setting's precise effect on a particular request or response path should be verified against the installed SillyTavern version.

### Practical diagnostic

For formatting problems:

1. Capture the raw inference response.
    
2. Inspect the text received by the SillyTavern backend.
    
3. Inspect the content delivered to the frontend.
    
4. Inspect the stored message representation.
    
5. Compare it with the rendered output.
    
6. Review relevant formatting settings.
    

The first point at which the text changes identifies the area requiring investigation.

## 9. Debugging by Pipeline Boundary

A reliable debugging strategy is to examine each boundary independently.

|Symptom|First area to investigate|
|---|---|
|Connection refused|Network and Ollama listener|
|HTTP error|Endpoint, authentication and request structure|
|Backend rejects parameter|API compatibility and parameter forwarding|
|Model returns no visible text|Reasoning configuration and response fields|
|Output truncates unexpectedly|Output limits, stop conditions and parsing|
|Streaming fails|Stream handling and incremental parsing|
|Missing paragraph spacing|Formatting and frontend processing|
|Incorrect character behaviour|Prompt construction and effective context|
|Preset regression|Final request comparison|

This table identifies starting points, not definitive diagnoses.

A single symptom can have several causes.

## 10. Inspecting the Actual Request

The most useful debugging artefact is the final request sent to the inference backend.

It should reveal:

- Endpoint and HTTP method
    
- Model identifier
    
- Prompt or ordered messages
    
- Generation parameters
    
- Reasoning configuration
    
- Streaming mode
    
- Stop sequences
    
- Context settings
    

Sensitive content should be removed before storing or sharing traces.

### Why frontend settings are insufficient

A frontend setting may be transformed or removed before transmission.

A backend may also apply defaults that are not explicitly represented in the frontend.

Consequently, comparing screenshots of SillyTavern settings is less reliable than comparing actual outbound requests.

## 11. Implications for the SillyTavern Prompt Workbench

The SillyTavern Prompt Workbench should distinguish at least three observability levels.

### Level 1 — Final request capture

Capture the actual outbound request immediately before transmission.

This establishes what SillyTavern sent.

### Level 2 — Response capture

Capture the raw inference response and the processed result returned to the frontend.

This establishes whether content changed during backend handling.

### Level 3 — Provenance tracing

Trace how prompt components and parameters entered the request.

This requires instrumentation earlier in the assembly pipeline.

It is more complex than simply capturing the final request.

### Design principle

**Observe the real SillyTavern pipeline rather than recreating it in a separate application.**

A reconstructed prompt may look plausible while differing from the request actually sent.

For the initial Workbench implementation, accurate final-request capture is the most valuable foundation.

## 12. Recommended Investigation Procedure

When diagnosing an unexpected response:

1. Record the SillyTavern version and API mode.
    
2. Record the model, backend and relevant versions.
    
3. Reproduce the issue using a controlled prompt.
    
4. Capture the final outbound request.
    
5. Confirm the backend accepts the request.
    
6. Inspect the raw response.
    
7. Compare raw and processed content.
    
8. Identify the first pipeline stage where behaviour diverges.
    
9. Make the smallest relevant change.
    
10. Repeat the test and document the result.
    

Avoid changing sampling parameters, presets and backend patches simultaneously.

Doing so obscures the cause of any improvement or regression.

## Related Notes

- [[SillyTavern Prompt Construction and Presets]]
    
- [[SillyTavern Docker Patching and Maintenance]]
    
- [[LLM Generation Parameters — A Practical Guide]]
    
- [[Quantisation, VRAM and Context]]
    
- [[Local Coding Agents]]
    

## Key Takeaway

**A model response passes through several distinct processing stages before appearing in SillyTavern.**

Unexpected output may originate in prompt assembly, request translation, inference, response extraction or frontend formatting.

Reliable debugging means finding the first boundary where the expected behaviour changes.

The foundation is simple: **capture what was actually sent, capture what was actually returned, and avoid guessing about the stages in between.**