---

type: reference  
status: active  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- comfyui-desktop
    
- comfyui-manager
    
- custom-nodes
    
- image-generation
    
- troubleshooting
    

---
## Overview

This article documents two related outcomes from troubleshooting ComfyUI Desktop:

1. Investigation of a ComfyUI-Manager security-policy restriction encountered while updating a custom-node extension.
    
2. Establishment of project-wide engineering instructions to improve version accuracy, troubleshooting efficiency, workflow continuity, and prompt collaboration.
    

The underlying objective is to avoid repeated investigations caused by confusing ComfyUI Desktop with standalone installations, relying on outdated extension documentation, or assuming that independently versioned components behave identically.

The project standards apply to future ComfyUI troubleshooting, workflow development, and image/video prompt engineering.

## 1. Environment Baseline

The environment recorded during the discussion was:

|Component|Version or configuration|
|---|---|
|Operating system|Windows 11|
|Installation type|ComfyUI Desktop|
|ComfyUI Core|v0.40.0|
|ComfyUI Desktop|Exact version not established|
|ComfyUI-Manager|Integrated/current as of 15 October 2026; exact version not established|
|Python|3.13.12|
|PyTorch|2.12.1+cu130|
|GPU|NVIDIA RTX 5070 Ti|
|VRAM|16 GB|
|System RAM|32 GB DDR5|
|Custom nodes|Managed through ComfyUI-Manager|

Known startup arguments included:

```
--listen 0.0.0.0
--enable-manager
--enable-assets
```

**Version caveat:** This is a historical baseline from 14–15 October 2026, not a guarantee of the currently installed versions.

ComfyUI Core, Desktop, Manager, and custom-node packages can update independently.

## 2. ComfyUI-Manager Security-Policy Error

### Observed problem

On 15 October 2026, updating an extension through ComfyUI-Manager produced the following error:

```
ERROR: To use this action, security_level must be `normal or below`, and network_mode must be set to `personal_cloud`. Please contact the administrator.
Reference: https://github.com/ltdrdata/ComfyUI-Manager#security-policy
```

The attempted operation was an update of:

```
Extension: comfyui-cyberdelia-metadata
Installed version: 2.2.1
Target version: 2.2.4
Operation: Update through ComfyUI-Manager
```

### Diagnosis

The error originated from ComfyUI-Manager's security-policy enforcement.

The update action was blocked because the Manager security configuration did not satisfy the permissions required for the operation.

The error does not, by itself, demonstrate:

- An incompatibility between the extension and ComfyUI Core.
    
- A broken extension installation.
    
- A Python dependency failure.
    
- A corrupted workflow.
    
- A failure in the extension's own code.
    

The observed failure was an authorization/configuration restriction.

### Configuration location

The user identified the Manager configuration file under:

```
users/_manager/config.ini
```

The exact path spelling and its relationship to the active Desktop user directory were not independently established in the conversation.

Other ComfyUI installations and Manager releases may use different configuration locations.

**Rule:** Confirm the configuration file actually loaded by the running Manager rather than assuming a conventional installation path.

### Required configuration

The error explicitly identified these requirements:

```
[default]
security_level = normal
network_mode = personal_cloud
```

The values were subsequently checked against contemporary ComfyUI Desktop and Manager documentation during the discussion.

The investigation established that `personal_cloud` was a supported network-mode value in the relevant contemporary Manager implementation.

This was significant because earlier documentation and older Manager versions used different network-mode conventions.

### Recommended resolution

1. Identify the active Manager `config.ini`.
    
2. Preserve a copy of the existing configuration.
    
3. Inspect the `[default]` section.
    
4. Verify the installed Manager version supports the required values.
    
5. Configure the two settings if they differ.
    
6. Restart the relevant ComfyUI processes so the configuration is reloaded.
    
7. Retry the extension update.
    
8. Verify that the update completes and that the intended extension version is installed.
    

The proposed settings were:

```
[default]
security_level = normal
network_mode = personal_cloud
```

These lines describe the required values, not a replacement for the entire configuration file.

### Resolution status

**The proposed fix was identified, but successful application was not confirmed in this conversation.**

No subsequent update result or installed-version confirmation was provided.

Therefore, the incident should not be documented as conclusively resolved.

### Network security implications

The ComfyUI instance was configured with:

```
--listen 0.0.0.0
```

This allows ComfyUI to listen on all available network interfaces rather than restricting connections to localhost.

Combining network accessibility with Manager permissions that allow extension installation increases the potential impact of unauthorized access.

Recommended precautions:

- Restrict network access to trusted devices.
    
- Avoid exposing the instance directly to the public internet.
    
- Use appropriate firewall or network-level restrictions.
    
- Do not lower security settings beyond those required for the intended operation.
    
- Reevaluate Manager permissions if remote access requirements change.
    

The `personal_cloud` setting should not be treated as a substitute for network access controls.

### Relevant upstream references

- [ComfyUI-Manager — Security Policy](https://github.com/ltdrdata/ComfyUI-Manager#security-policy)
    
- [ComfyUI Desktop repository](https://github.com/Comfy-Org/Comfy-Desktop)
    
- [ComfyUI-Manager repository](https://github.com/Comfy-Org/ComfyUI-Manager)
    
- [ComfyUI-Manager issue #2496](https://github.com/Comfy-Org/ComfyUI-Manager/issues/2496)
    

These references provide starting points for version-matched verification. They do not replace evidence from the locally installed implementation.

## 3. Why the Investigation Became Unnecessarily Complicated

The initial troubleshooting mixed assumptions about different generations of ComfyUI-Manager.

In particular:

- Older Manager documentation did not necessarily describe the current Desktop-integrated implementation.
    
- Configuration paths were initially inferred from historical installation layouts.
    
- The validity of `personal_cloud` was questioned before checking current upstream sources.
    
- The installed Manager implementation was not identified before configuration advice was offered.
    

This caused unnecessary uncertainty around a relatively specific error message.

The important lesson is that **version-dependent behaviour must be verified before presenting configuration advice as authoritative**.

This applies equally to ComfyUI Core, Desktop, Manager, and individual custom-node packages.

## 4. Mandatory Version-Awareness Standards

### Identify components independently

Always distinguish between:

- ComfyUI Core
    
- ComfyUI Desktop
    
- ComfyUI-Manager
    
- Individual custom-node packages
    
- Python and PyTorch environments
    
- Model-specific dependencies
    

Do not infer one component's version from another.

### Source verification

Before recommending version-dependent changes:

1. Establish the installed component version where necessary.
    
2. Check current official documentation, source code, release notes, or relevant upstream issues.
    
3. Prefer documentation matching the installed release.
    
4. Compare upstream information against local logs and configuration.
    
5. Identify discrepancies rather than silently assuming the newest documentation applies.
    

### Evidence priority

For an installed environment, prioritize:

1. Actual runtime behaviour and reproducible results.
    
2. Startup logs and error messages.
    
3. Installed source code and package metadata.
    
4. Version-matched upstream documentation.
    
5. Historical documentation and community reports.
    

Historical information can explain changes but must not automatically be treated as current behaviour.

### Configuration validation

Before recommending a configuration change, verify:

- The configuration key exists.
    
- The proposed value is accepted.
    
- The setting applies to the installed component.
    
- The correct configuration file has been identified.
    
- Whether the change requires a restart.
    
- Any security or compatibility implications.
    

Never present speculative configuration values as confirmed fixes.

## 5. Troubleshooting Standards

### Start with the actual failure

Identify the failing component, operation, error, and relevant versions.

Avoid diagnosing unrelated parts of the workflow without evidence.

### Prefer minimal changes

Recommend the smallest justified intervention.

Avoid unnecessary:

- Reinstallations.
    
- Workflow rebuilds.
    
- Checkpoint replacements.
    
- Configuration resets.
    
- Extension removals.
    
- Security-policy changes.
    

### Preserve working behaviour

Previously successful configurations and workflow components should be treated as known-good unless evidence indicates otherwise.

When investigating regressions, consider recent updates to Core, Desktop, Manager, and custom nodes.

### Incremental testing

Change one relevant variable at a time when practical.

Preserve seeds and generation parameters when comparing workflow behaviour.

Explain what each change is intended to test and what result would confirm or reject the hypothesis.

### Distinguish evidence from hypotheses

Clearly identify whether a conclusion is:

- Directly observed.
    
- Verified against upstream implementation.
    
- Supported by testing.
    
- A plausible but unverified explanation.
    

Do not describe suspected regressions or incompatibilities as established facts.

### Avoid unnecessary investigations

If the failure has a specific, documented solution, prioritize that solution.

Broaden the investigation only when the available evidence justifies doing so.

## 6. Workflow Architecture and Custom Nodes

### Preserve established architecture

Existing workflows should be treated as the starting point for modifications.

Do not propose structural changes unless necessary to achieve the requested result or resolve a demonstrated problem.

### Understand the complete processing path

When relevant, consider:

- Model and checkpoint loading.
    
- CLIP and text conditioning.
    
- LoRA loading and stacking.
    
- Sampler configuration.
    
- VAE encoding and decoding.
    
- Detailers and inpainting.
    
- Switches and conditional execution.
    
- Upscaling and final output nodes.
    

### Identify custom-node dependencies

Distinguish native ComfyUI nodes from third-party implementations.

Verify the applicable package and installed version before diagnosing node-specific behaviour.

### Preserve reproducibility

For controlled comparisons, preserve relevant generation settings such as:

```
Seed
Sampler
Scheduler
Steps
CFG
Denoise
Checkpoint
LoRA weights
Resolution
Conditioning
```

Change only the variables required by the experiment where practical.

### Model behaviour versus workflow behaviour

A prompt-interpretation issue is not automatically a workflow defect.

Likewise, a node execution failure is not necessarily caused by the model.

Diagnosis should follow the observed processing path.

## 7. Prompt Creation and Revision Standards

### Mandatory complete-prompt rule

**Whenever creating, revising, refining, or troubleshooting a ComfyUI prompt, always provide the complete updated prompt, ready to copy and paste.**

This applies to image and video generation models, including:

- Illustrious and Illustrious-derived checkpoints.
    
- Krea 2 and Krea 2 Turbo.
    
- SDXL and SDXL-derived checkpoints.
    
- Flux and Flux-derived models.
    
- Anima.
    
- Other diffusion and generative architectures.
    

Never provide only a modified section, isolated replacement tags, or instructions to insert a fragment into an earlier prompt.

### Preserve existing content

When revising a prompt:

- Retain relevant existing content.
    
- Preserve intentional structure and ordering.
    
- Keep established trigger words and weights where applicable.
    
- Avoid silently removing unrelated details.
    
- Make targeted changes rather than rewriting successful sections unnecessarily.
    

### Respect model-specific syntax

Different model families may require different prompting conventions.

Examples include:

- Natural-language descriptions.
    
- Danbooru-style tags.
    
- Model-specific weighting syntax.
    
- LoRA trigger words.
    
- Positive and negative conditioning.
    

Do not assume that techniques suitable for one model family are automatically transferable to another.

### Positive and negative prompts

If both prompts are modified, provide both in full.

If only one is changed, provide that complete updated prompt without unnecessarily repeating the unchanged one.

### Avoid prompt inflation

Do not introduce unnecessary:

- Quality tags.
    
- Descriptive repetition.
    
- Conflicting instructions.
    
- Weights without a clear purpose.
    
- Unrequested artistic or compositional changes.
    

Each addition should serve a specific objective.

### Respect enhancement workflows

Distinguish between:

- The original user prompt.
    
- Generated Danbooru tags.
    
- Model-enhanced prompts.
    
- Final conditioning passed to the generation model.
    

These are separate stages and should not be treated as interchangeable.

### Use actual results

Prompt refinement should be guided by the user's generated images, observed model behaviour, and desired outcome.

Explain significant changes separately from the complete updated prompt.

## 8. Collaboration Standards

The project assumes substantial software engineering experience.

Communication should therefore prioritize precision, evidence, and practical relevance over generic introductory explanations.

### Maintain continuity

Use established project decisions, previous troubleshooting results, and confirmed workflow configurations.

Do not restart investigations unnecessarily or repeatedly request information already established unless it may have changed.

### Explain mechanisms

When useful, explain why a setting or change affects the result.

Understanding the underlying mechanism is preferable to relying on unexplained recipes.

### Support controlled experimentation

Where behaviour is uncertain, propose experiments that isolate meaningful variables and preserve reproducibility.

### Correct errors explicitly

When an earlier recommendation is found to be outdated or incorrect, acknowledge the mistake and replace it with verified information.

### Avoid unnecessary complexity

Simple questions should receive direct answers.

Do not expand a targeted problem into an unrelated redesign or broad troubleshooting exercise.

## 9. Consolidated Engineering Principles

The following principles govern future ComfyUI collaboration:

1. Verify the actual installation environment.
    
2. Identify the relevant installed component versions.
    
3. Consult current, version-matched upstream evidence.
    
4. Prefer observed behaviour over assumptions.
    
5. Preserve existing working configurations and workflows.
    
6. Investigate the specific failure before exploring unrelated causes.
    
7. Recommend the smallest justified change.
    
8. Keep experiments reproducible.
    
9. Distinguish verified facts from hypotheses.
    
10. Always provide complete, immediately usable revised prompts.
    

## 10. Outstanding Work

The following items remained unverified at the conclusion of the discussion:

- The exact ComfyUI Desktop version.
    
- The exact installed ComfyUI-Manager version.
    
- The canonical configuration path loaded by the active Manager instance.
    
- Whether the proposed security-policy changes were applied.
    
- Whether `comfyui-cyberdelia-metadata` was successfully updated to version 2.2.4.
    

These should be resolved using installed-version evidence if they become relevant to subsequent troubleshooting.

## Conclusion

The Manager security-policy incident illustrated the importance of distinguishing installation types, verifying version-dependent settings, and following the evidence presented by the actual error.

The resulting project standards establish a consistent approach to ComfyUI development and troubleshooting:

**Verify the environment. Confirm the versions. Consult applicable upstream evidence. Preserve what already works. Make the smallest justified change.**

For prompt engineering:

**Always provide the complete updated prompt, never merely the section that changed.**

These standards are intended to reduce repeated investigations, prevent avoidable configuration mistakes, and maintain continuity across future workflow development.