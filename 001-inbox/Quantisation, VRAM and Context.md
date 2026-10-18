---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- ai
    
- llm
    
- ollama
    
- quantisation
    
- gguf
    
- vram
    
- context-window
    

---
## Purpose

This note explains the practical relationship between **model size, quantisation, GPU memory, context length and inference performance** when running local large language models.

The objective is to make informed model-selection decisions without assuming that a model fitting on disk will necessarily fit into GPU memory or run efficiently.

The examples focus on Ollama and consumer NVIDIA GPUs, particularly the RTX 5070 Ti with 16 GB VRAM.

## 1. The Local Inference Memory Budget

Running an LLM requires memory for more than its model weights.

The main components are:

|Component|Purpose|
|---|---|
|Model weights|The model's learned parameters|
|KV cache|Stores attention keys and values for processed tokens|
|Runtime overhead|Buffers, execution workspace and backend allocations|
|GPU overhead|Display usage, driver allocations and other GPU applications|

A useful approximation is:

**Required memory ≈ model weights + KV cache + runtime overhead**

The model's file size is therefore only a starting point when estimating memory requirements.

A model that occupies 14 GB on disk may require substantially more than 14 GB of available VRAM to operate entirely on the GPU.

## 2. Quantisation

Quantisation reduces the numerical precision used to represent model weights.

Its principal benefit is reducing memory consumption, often allowing larger models to run on hardware that could not accommodate their original precision.

The trade-off is potential loss of model accuracy or output quality.

### Common formats

|Format|General characteristics|
|---|---|
|FP16 / BF16|High precision, relatively large memory requirements|
|FP8|Lower memory requirements with comparatively modest precision reduction|
|Q8|Eight-bit quantised weights, generally high fidelity|
|Q6|Moderate compression with relatively small quality trade-offs|
|Q5|Useful balance of quality and memory consumption|
|Q4|Common choice for constrained consumer hardware|
|Q3|More aggressive compression, potentially useful for larger models|
|Q2|Very aggressive compression, with greater quality risk|

Actual quality depends on the model, quantisation method and workload.

**A larger model at lower precision is not automatically better than a smaller model at higher precision.**

### GGUF quantisation names

GGUF files commonly use names such as:

- `Q4_K_M`
    
- `Q5_K_M`
    
- `Q3_K_XL`
    
- `IQ4_XS`
    

These identify specific quantisation schemes, not merely the number of bits per parameter.

The suffixes and prefixes reflect implementation-specific approaches to balancing storage, computational cost and fidelity.

For example, two nominally four-bit quantisations may differ in file size, quality and runtime behaviour.

Unsloth's `UD-` variants are associated with its dynamic quantisation approach.

Do not assume that similarly named quantisations have identical behaviour across model families or inference engines.

## 3. GPU Residency and CPU Offloading

Ideally, a model and its working memory remain in GPU VRAM during inference.

This allows computation to take advantage of the GPU's parallel processing capability and memory bandwidth.

When insufficient VRAM is available, some inference engines can place portions of the model in system RAM and execute some layers on the CPU.

This is commonly called **CPU offloading** or partial GPU offloading.

### Fully GPU-resident

Advantages:

- Generally higher inference throughput
    
- Avoids CPU execution of offloaded layers
    
- Reduced dependence on system RAM bandwidth
    

### Partially offloaded

Advantages:

- Enables models that exceed available VRAM
    
- Allows experimentation with larger or higher-precision models
    

Disadvantages:

- Often significantly slower
    
- Greater CPU and system RAM involvement
    
- Performance becomes dependent on the model, offload split and hardware
    

Offloading is not inherently a failure. It is a deliberate compromise between model capability and performance.

However, for interactive roleplay or coding agents that make repeated tool calls, additional latency can become particularly noticeable.

## 4. The Context Window and KV Cache

The context window determines how much tokenised information the model can process within its configured context limit.

That information may include:

- System and developer instructions
    
- Character cards and world information
    
- Conversation history
    
- Retrieved documents or source code
    
- Tool results
    
- Newly generated tokens
    

During inference, the model maintains a **key-value cache**, or KV cache, to avoid repeatedly recomputing attention information for earlier tokens.

### Why context consumes memory

KV cache requirements generally grow with the number of tokens retained.

The precise memory cost depends on:

- Model architecture
    
- Number of layers
    
- KV attention heads and head dimensions
    
- Cache precision
    
- Configured context length
    
- Runtime cache-management strategy
    

A model supporting a very large theoretical context window does not mean that the full window is practical on a particular GPU.

### Context length versus useful context

A larger context window is not automatically better.

It may increase memory consumption and processing cost while introducing more irrelevant information into the prompt.

For coding agents, excessive tool output and repeated source-file reads can make a large context window expensive without making the agent more effective.

The goal is **sufficient relevant context**, not the largest possible context.

## 5. Quantisation and Context Compete for VRAM

On a memory-constrained GPU, model weights and KV cache compete for the same finite resource.

A higher-quality quantisation may leave insufficient memory for a useful context window.

A smaller quantisation may permit a longer context or greater runtime headroom.

For example:

|Configuration|Likely trade-off|
|---|---|
|Smaller model, higher precision|More VRAM available for context|
|Larger model, aggressive quantisation|Potentially stronger model capability, reduced precision|
|Larger model, higher precision|May require CPU offloading|
|Smaller quantisation, shorter context|More GPU headroom and potentially better responsiveness|

The optimum depends on the workload.

For an interactive coding agent, a slightly smaller or more aggressively quantised model that remains GPU-resident may outperform a larger, partially offloaded model in overall task completion time.

## 6. Practical Example — RTX 5070 Ti 16 GB

The local AI workstation uses an NVIDIA RTX 5070 Ti with approximately 16 GB VRAM.

This is capable hardware for local inference, but 20B–30B-class models require careful memory management.

### Qwen3.8 27B GGUF

The Unsloth Qwen3.8 27B GGUF release provides several quantisations.

The following figures were discussed as approximate model file sizes:

|Quantisation|Approximate file size|Practical consideration|
|---|---|---|
|UD-IQ3_S|12.0 GB|More room for context and overhead|
|UD-Q3_K_XL|13.1 GB|Candidate starting point|
|UD-IQ4_XS|14.3 GB|Tight but worth testing|
|UD-Q4_K_S|15.4 GB|Very limited remaining VRAM|
|UD-Q4_K_M|16.5 GB|Exceeds nominal VRAM before runtime overhead|

These are **file sizes, not measured total GPU allocations**.

They should not be interpreted as guarantees of full GPU residency.

The exact memory footprint depends on the inference engine, context configuration and other runtime allocations.

### Suggested evaluation sequence

1. Begin with a quantisation that leaves meaningful VRAM headroom.
    
2. Configure a modest context window, such as 8K.
    
3. Load the model and inspect actual GPU usage.
    
4. Verify whether inference is fully GPU-resident.
    
5. Measure responsiveness and generation throughput.
    
6. Increase context to 16K if memory allows.
    
7. Compare a higher-quality quantisation only after establishing the baseline.
    

This is preferable to loading the largest available quantisation and immediately requesting the maximum advertised context.

## 7. Monitoring Ollama

Ollama provides useful commands for inspecting running models.

### Check loaded models

```
ollama ps
```

The processor information can indicate whether a model is running entirely on the GPU or is partially offloaded.

### Monitor NVIDIA GPU memory

```
nvidia-smi
```

For repeated monitoring:

```
nvidia-smi -l 1
```

Useful observations include:

- GPU memory usage
    
- GPU utilisation
    
- Power consumption
    
- Whether allocations change during prompt processing or generation
    

### Distinguish loading from inference

Model loading, prompt evaluation and token generation are separate performance phases.

A model may load successfully yet generate slowly.

Similarly, the first response may be slower than subsequent responses because model weights must be loaded into memory.

Record performance after warm-up as well as initial loading behaviour.

## 8. Evaluating Models for Coding Agents

For coding agents, raw token-generation speed is only one factor.

A useful evaluation considers:

|Criterion|Why it matters|
|---|---|
|Code understanding|Ability to follow existing architecture|
|Tool calling|Reliable interaction with files and commands|
|Instruction adherence|Respecting task scope and constraints|
|Context management|Avoiding unnecessary repeated information|
|Debugging ability|Identifying causes rather than guessing fixes|
|Generation speed|Responsiveness during iterative tool use|
|Memory footprint|Whether the desired context fits in VRAM|
|Task completion|Whether changes actually work and pass tests|

A model generating more tokens per second may still complete tasks more slowly if it makes more mistakes or requires repeated attempts.

For our SillyTavern Prompt Workbench project, a representative benchmark would be:

> Inspect the SillyTavern source code, identify the request-assembly path, document the relevant call chain, and propose instrumentation points without changing production code.

This tests repository navigation, comprehension, instruction adherence and tool use.

It is more meaningful than asking the model to generate an isolated function.

## 9. A Practical Model-Selection Strategy

Model selection should follow the workload rather than the parameter count alone.

### Interactive roleplay

Prioritise:

- Response quality and character consistency
    
- Acceptable generation latency
    
- Sufficient conversation context
    
- Reliable formatting and stopping behaviour
    

### Coding agents

Prioritise:

- Tool-call reliability
    
- Repository understanding
    
- Accurate edits
    
- Sensible context consumption
    
- Overall task completion time
    

### Long-document analysis

Prioritise:

- Effective context capacity
    
- Retrieval and information selection
    
- Prompt-processing performance
    
- Memory efficiency
    

Different workloads may justify different models and quantisations on the same hardware.

## 10. Common Misconceptions

**“If the GGUF file fits in VRAM, the model will fit.”**

Not necessarily. KV cache and runtime allocations also require memory.

**“A model with a 256K context window should always run at 256K.”**

No. Advertised context capacity and practical local inference capacity are different.

**“Higher quantisation precision always produces better results.”**

Not necessarily for a given workload. Model capability and quantisation quality both matter.

**“CPU offloading makes a model unusable.”**

No, but it may make interactive use substantially slower.

**“A 27B model must outperform a 12B model.”**

No. Training, architecture, fine-tuning, quantisation and task requirements can outweigh parameter count.

**“The fastest model is the best coding agent.”**

No. End-to-end task completion is the more meaningful measure.

## 11. Recommended Benchmark Record

|Property|Value|
|---|---|
|Model||
|Quantisation||
|Ollama version||
|Context window||
|GPU memory used||
|GPU/CPU residency||
|Prompt-processing speed||
|Generation speed||
|Time to complete task||
|Task result||
|Observed issues||

Keep the prompt, tool permissions and evaluation criteria consistent when comparing models.

For reliable comparisons, repeat tests and distinguish measured results from subjective impressions.

## Related Notes

- [[LLM Generation Parameters — A Practical Guide]]
    
- [[SillyTavern Prompt Construction and Presets]]
    
- [[SillyTavern Request and Response Pipeline]]
    
- [[Local Coding Agents]]
    

## Key Takeaway

**Local LLM performance is a balancing act between model capability, quantisation, GPU memory, context length and workload requirements.**

The best configuration is not necessarily the largest model or the highest-quality quantisation.

It is the configuration that provides sufficient capability, useful context and acceptable responsiveness while remaining within the hardware's practical limits.