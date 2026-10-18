---

type: troubleshooting  
status: draft  
created: 2026-10-18  
source: ""  
tags:

- comfyui
    
- comfyui-impact-pack
    
- workflow-compatibility
    
- impact-switch
    
- debugging
    

---
## Overview

Following updates to the ComfyUI environment, two problems were observed involving workflows using ComfyUI Impact Pack:

1. Workflows failing to open.
    
2. Unexpected behaviour from Impact Switch nodes.
    

The workflow-loading problem was resolved locally by applying an upstream patch.

The switch problem remained unresolved after that patch.

Although the symptoms may be related, a common underlying cause has not been established.

## Environment

Last confirmed environment during the investigation:

|Component|Version or configuration|
|---|---|
|Operating system|Windows 11|
|Installation|ComfyUI Desktop|
|ComfyUI Core|v0.40.0|
|Python|3.13.12|
|PyTorch|2.12.1+cu130|
|GPU|NVIDIA RTX 5070 Ti, 16 GB|
|ComfyUI Impact Pack|Exact installed version not established|
|ComfyUI-Manager|Integrated with ComfyUI Desktop|

The environment is a managed ComfyUI Desktop installation, not a conventional portable or standalone installation.

Version numbers describe the known historical environment and should be rechecked before further investigation.

## Issue 1 — Workflows Failing to Open

### Observed Behaviour

Some workflows using Impact Pack failed to open entirely.

This was a workflow-loading problem rather than merely a generation-time failure.

### Resolution

A patch found through an upstream GitHub issue or pull request was applied locally.

**Confirmed result:** Workflow loading worked again after the patch.

The exact patch identifier and commit have not been independently established.

During the discussion, PR #1223 was mentioned initially, followed by a possible reference to PR #1228. Neither identifier should be treated as the confirmed applied patch without inspecting the actual diff or local source.

Relevant upstream repository:

[https://github.com/ltdrdata/ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack)

### Status

**Locally resolved.**

The patch was applied manually, so the local installation may differ from an unpatched upstream release.

Whether the fix has been incorporated into a subsequent installed release has not been established.

## Issue 2 — Impact Switch Behaviour

### Observed Behaviour

Impact Switch nodes exhibited inconsistent behaviour.

The following observations were reported:

|Test|Result|
|---|---|
|Existing normal Impact Switch|Failed|
|Inverse switch|Worked|
|Newly created normal Impact Switch|Worked|
|Duplicate of a working switch|Failed|
|Original switch after workflow-loading patch|Still failed|

The particularly significant observation is that a newly created switch worked while a duplicate of a working switch did not.

This suggests that the failure may depend on something beyond the apparent switching logic.

### Possible Causes

The following remain hypotheses rather than confirmed diagnoses:

- Node serialization or deserialization differences
    
- Widget configuration or persisted node properties
    
- Changes in frontend node handling
    
- Impact Pack compatibility with the updated ComfyUI environment
    
- Differences between newly created and duplicated node state
    

The timing of the failure following a ComfyUI update makes a compatibility regression plausible, but does not establish its cause.

There is insufficient evidence to attribute the problem definitively to ComfyUI Core, the frontend or Impact Pack.

### Relationship to Workflow-Loading Patch

The locally applied patch restored workflow loading.

It did not restore the failing switch.

This establishes that the patch was insufficient to resolve both symptoms.

It does not prove that the two failures have completely independent root causes.

### Status

**Unresolved.**

A working alternative exists, allowing image-generation experiments to continue.

## Existing Workflow Architecture

The affected Krea 2 workflow includes:

- Base text-to-image generation
    
- Eye Detailer subgraph
    
- Manual inpainting route
    
- Impact Switch-based output routing
    
- Optional 2× upscaling
    

The routing architecture is intended to support different processing paths without requiring separate complete workflows.

The switch problem affects routing and convenient access to the desired outputs.

There is no confirmed evidence that the switch issue alters the underlying image-generation quality.

## Temporary Workaround — Preview Nodes

Strategically placed preview nodes allow intermediate outputs to be inspected without relying on the failing final routing path.

The approach is to connect preview nodes to relevant image outputs, such as:

- Base generation
    
- Eye Detailer output
    
- Manual inpainting output
    
- Other relevant processing stages
    

The preview nodes expose generated images for inspection while leaving the existing processing architecture largely unchanged.

### Advantages

- Avoids dependence on the failing switch for visual inspection.
    
- Preserves the existing generation and detailing pipeline.
    
- Allows continued prompt and workflow experimentation.
    
- Requires only small, reversible workflow changes.
    
- Avoids replacing working components unnecessarily.
    

### Limitations

- Does not repair the Impact Switch.
    
- Does not fully replace automated routing to the intended output node.
    
- Adds visual clutter to the workflow.
    
- May not provide the same file-saving behaviour as the original output route.
    

### Current Decision

Continue using preview nodes until a verified upstream fix becomes available or further investigation becomes worthwhile.

The workaround is sufficient for the current image-generation experiments.

## Further Investigation

No additional investigation is currently required to continue working.

If troubleshooting resumes, a useful controlled experiment would be to compare the serialized workflow data of:

1. A newly created, working Impact Switch.
    
2. A duplicated copy that fails.
    

Compare node type, widget values, properties, connections and any extension-specific metadata.

The purpose would be to identify a reproducible difference between working and failing nodes.

A second useful step would be to capture the relevant frontend console errors and execution logs.

These investigations should be performed on a copy of the workflow to preserve the working configuration.

## Update and Maintenance Considerations

Because the workflow-loading fix was applied locally, a future Impact Pack update may replace the patched files.

Before updating:

- Identify whether the installed release contains the workflow-loading fix.
    
- Preserve the known-good workflow.
    
- Check the relevant upstream issue or pull request.
    
- Avoid assuming that a new Impact Pack release is compatible with the installed Core and frontend versions.
    

A future release may resolve one issue without resolving the other.

## Conclusions

The investigation established two distinct observed failures.

**Workflow loading:** Resolved locally through an upstream patch.

**Impact Switch:** Still failing under specific conditions, including duplication of an otherwise working node.

The original switch issue remains undiagnosed.

Strategically placed preview nodes provide a practical, non-destructive workaround, allowing image generation and eye-detailing experiments to continue.

Further debugging is deferred until necessary.

The guiding decision is to preserve the existing working workflow and avoid unnecessary structural changes to compensate for an unresolved third-party compatibility problem.