---

type: reference  
status: active  
created: 2026-10-18  
source:  
tags:

- sillytavern
    
- stmb
    
- memory-books
    
- configuration
    
- openrouter
    

purpose: knowledge
---
## Purpose

Document the agreed SillyTavern Memory Books (STMB) configuration as a reference for future maintenance and troubleshooting.

The configuration has been tested successfully. Settings should not be changed solely to match STMB recommendations or presumed defaults.

This note distinguishes between:

- **Confirmed changes:** Settings deliberately modified during testing.
    
- **Configured values:** Settings recorded in the working configuration, without proof that they differ from factory defaults.
    
- **Recommendations:** Settings discussed but not independently confirmed as saved.
    

## 1. Environment

|Component|Configuration|
|---|---|
|Roleplay model|`google/gemma-4-31b-it`|
|Memory model|`google/gemma-4-26b-a4b-it`|
|Connection provider|OpenRouter|
|Character card|Seris|
|User persona|D3f@u1t|
|Chat type|Solo character chat|
|Memory storage|Dedicated STMB Memory Book|
|Narrator Mode|Not enabled|

The roleplay model portrays Seris and recurring NPCs within the same chat. Native SillyTavern group chat is not used.

STMB stores generated memories as SillyTavern World Info entries.

## 2. Memory Generation Profile

**Profile:** `Gemma 26B — Memories`

### 2.1 Model and Connection Settings

|Setting|Value|
|---|---|
|Model|`google/gemma-4-26b-a4b-it`|
|Temperature|`0.2`|
|Use reverse proxy|Enabled|
|Skip structured output and use plain-text completion|Disabled|
|Use ST's ChatCompletionService|Disabled|

A dedicated memory model and low temperature were configured to improve summary consistency.

The previous roleplay temperature of approximately `1.2` was considered a possible cause of inconsistent summaries. Because both model and temperature were changed, the individual contribution of each was not established.

Subsequent memory-generation testing produced a satisfactory summary.

### 2.2 Summary Settings

|Setting|Value|
|---|---|
|Memory Creation Method|Summary|
|Summary style|Detailed beat-by-beat narrative prose|
|Memory Title Format|`[000] - {{title}}`|
|Separate group and character prompts|Disabled|

### 2.3 Memory Entry Settings

|Setting|Value|
|---|---|
|Activation Mode|Normal|
|Insertion Position|`↑Char`|
|Insertion Order|Auto|
|Prevent Recursion|Disabled|
|Delay Until Recursion|Disabled|

Normal activation is intentional and should not be changed to Vectorized without a specific reason.

### 2.4 Default Profile Selection

|Setting|Value|
|---|---|
|Default profile|Gemma 26B — Memories|
|Always use default profile|Enabled|

This prevents memory generation from unintentionally inheriting the active roleplay model's settings.

## 3. Automatic Memory Generation

### 3.1 Summary Scheduling

|Setting|Value|
|---|---|
|Auto-Summary Interval|25 messages|
|Auto-Summary Buffer|2 messages|
|Previous Memories Count|2|

**Confirmed change:** Previous Memories Count increased from `0` to `2`.

Previous Memories Count controls how many earlier memories are provided to the summarizer as reference material. It does not limit the number of memories available for retrieval during roleplay.

The handover records the interval and buffer values but does not establish whether they differ from STMB defaults.

### 3.2 Automatic Message Hiding

|Setting|Value|
|---|---|
|Auto-hide through latest memory|Enabled|
|Messages left unhidden|2|

Hidden messages remain in the chat history. They are excluded from the ordinary visible prompt history but are not deleted.

## 4. Vector Storage and Retrieval

### 4.1 Vector Storage Configuration

|Setting|Value|
|---|---|
|Vector source|Local (Transformers)|
|Query Messages|2|
|Score Threshold|0.3|
|World Info retrieval|Enabled|
|Enabled for all entries|Enabled|
|World Info Max Entries|5|
|Include in World Info Scanning|Disabled|
|Chat-message vectorization|Disabled|
|File vectorization|Disabled|
|Vector summarization options|Disabled|

### 4.2 Confirmed Changes

**Chat-message vectorization: Enabled → Disabled**

Chat-message vectorization was disabled to isolate Memory Book retrieval from retrieval of historical chat messages, including hidden messages.

World Info vector retrieval remains enabled.

**Ordinary memory activation: Vectorized → Normal**

Ordinary scene memories were switched to Normal activation during testing.

With World Info vector retrieval enabled for all entries, Normal entries remain eligible for vector similarity retrieval as well as ordinary World Info activation.

### 4.3 Settings Already Enabled

The following were already enabled when inspected:

- World Info retrieval
    
- Enabled for all entries
    

They were retained, not newly enabled.

The handover does not establish whether the remaining recorded Vector Storage values differ from factory defaults.

### 4.4 Retrieval Test Results

|Test|Observation|
|---|---|
|Explicit reference to Seris's sunburn|Relevant memory activated|
|Some alternative phrasings|Memory did not activate|
|“How are your shoulders feeling now?”|Relevant sunburn and maintenance-tunnel context appeared|

The final test succeeded with chat-message vectorization disabled.

> [!warning] Retrieval attribution  
> The tests demonstrated that the combined retrieval configuration worked. They did not establish whether vector similarity, keyword activation, or other available context was individually responsible.

**Decision:** Retain Normal activation with World Info vector retrieval enabled. Do not add exhaustive synonym lists without a demonstrated retrieval problem.

## 5. Memory Books and NPC Lore

### 5.1 Separation of Responsibilities

|Component|Responsibility|
|---|---|
|STMB Memory Book|Events, discoveries, promises, relationships and narrative continuity|
|Story lorebook|Stable NPC identities, traits and established facts|
|STMB Additional Context|Reference information provided to the summarizer|

The story lorebook is associated with Seris through Character Library.

### 5.2 NPC Reference Context

|Setting|Value|
|---|---|
|Additional Context|Seris - NPC References|
|NPC Side Prompt tracker|Not configured|

The Additional Context selection is used during memory summarization. It is separate from story-lore retrieval during roleplay.

### 5.3 Raxen

Raxen is a recurring maintenance technician with a faulty voice amplifier that sometimes stutters or repeats words.

|Setting|Value|
|---|---|
|Lore entry title/key|Raxen|
|Activation Mode|Normal|

Raxen's name was generated by the roleplay model despite an earlier naming instruction. The memory summary correctly recorded the scene as it occurred.

This was a roleplay instruction-following issue, not evidence of a memory-summary error.

## 6. Arc Consolidation

### 6.1 Consolidation Settings

|Setting|Value|
|---|---|
|Tier|1 — Arc|
|Preset|Multi-Consolidation Analysis|
|Maximum entries per pass|15|
|Token Budget|131072|
|Minimum eligible memories|5|
|Activation Mode|Vectorized|
|Insertion Position|`↑Char`|
|Insertion Order|Auto|
|Delay Until Recursion|Disabled|
|Disable selected source entries after creating summaries|Enabled|

The token budget is a batching ceiling, not a target for token consumption.

### 6.2 Recommendations Not Confirmed as Saved

|Setting|Recommended value|
|---|---|
|Automatic summary attempts|2, reduced from 10|
|Prevent Recursion|Enabled|

The handover records these recommendations but does not independently confirm their final saved state.

### 6.3 Consolidated Scene Memories

The following five scenes were consolidated:

1. Station Maintenance Tunnels
    
2. The Junction Override
    
3. The Starlight Ambush
    
4. The Void's Breach
    
5. The Shadow of Val Daran
    

**Arc title:** The Void's Shadow on Starlight Runner

The consolidation preview was accepted and saved.

The five source entries were disabled after successful consolidation but remain stored.

### 6.4 Narrative Continuity

The Arc establishes that Seris and D3f@u1t escaped the desert world aboard the Starlight Runner, survived an ambush and a Void attack, and identified Val Daran as a former Jedi researcher involved in attempts to integrate the Void into consciousness.

The Void has mapped their spiritual resonance, and the Jedi Council has confirmed Val Daran's involvement.

**Current story position:** The Starlight Runner is still in hyperspace travelling toward Coruscant.

Val Daran has been repelled, not defeated.

## 7. Change Record

### 7.1 Confirmed Changes and Deliberate Decisions

|Area|Change or decision|
|---|---|
|Memory model|Dedicated Gemma 4 26B A4B configured|
|Memory temperature|Set to 0.2|
|Default profile|Dedicated memory profile selected and enforced|
|Previous Memories Count|0 → 2|
|Scene-memory activation|Vectorized → Normal|
|Chat-message vectorization|Enabled → Disabled|
|NPC references|Additional Context configured|
|Story lore|Separate NPC lorebook associated through Character Library|
|Consolidation|Five scene memories consolidated into one Arc|
|Source memories|Disabled following accepted consolidation|

Not every deliberate configuration decision is a proven deviation from STMB factory defaults.

### 7.2 Recorded Working Values

The following are recorded as part of the working baseline, without a confirmed default comparison:

|Setting|Value|
|---|---|
|Auto-Summary Interval|25|
|Auto-Summary Buffer|2|
|Auto-hide messages left unhidden|2|
|Vector source|Local (Transformers)|
|Query Messages|2|
|Score Threshold|0.3|
|World Info Max Entries|5|
|Include in World Info Scanning|Disabled|

## 8. Troubleshooting

### 8.1 Configuration Checklist

- Dedicated memory profile is `Gemma 26B — Memories`.
    
- Memory model is `google/gemma-4-26b-a4b-it`.
    
- Temperature is `0.2`.
    
- Always use default profile is enabled.
    
- Auto-Summary Interval is `25`.
    
- Auto-Summary Buffer is `2`.
    
- Previous Memories Count is `2`.
    
- Auto-hide leaves `2` messages unhidden.
    
- Ordinary scene memories use Normal activation.
    
- World Info vector retrieval is enabled.
    
- Enabled for all entries is enabled.
    
- Chat-message vectorization is disabled.
    
- The correct Memory Book is bound to the chat.
    
- The relevant memory entry is enabled.
    
- `Seris - NPC References` is selected as Additional Context.
    
- The consolidated Arc remains enabled.
    
- The five consolidated source scenes remain disabled.
    

### 8.2 Diagnostic Sequence

When a memory appears to be missing, investigate the following stages independently.

**1. Generation**

Was the summary produced correctly, using the intended model and profile?

**2. Storage**

Was the summary saved in the correct Memory Book, with the expected entry settings?

**3. Retrieval**

Was the saved entry activated and included in the roleplay prompt?

Successful generation and storage do not guarantee subsequent retrieval.

## 9. Current Status and Next Steps

The working configuration has demonstrated:

- Successful memory generation using the dedicated profile
    
- Successful contextual recall
    
- Successful consolidation of five scene memories into an Arc
    
- Correct disabling of source entries after consolidation
    

The next useful validation is to continue the story, allow new scene memories to accumulate, and observe whether references to Val Daran or the Starlight Runner activate the consolidated Arc appropriately.

No further configuration changes are currently required.

## References

- STMB configuration and testing handover, 2026-10-18
    
- STMB AI Reference Manual, v8.5.0, 2026-08-01
    
- STMB summary profile screenshot, 2026-10-18