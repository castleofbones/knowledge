---
type: reference
status: draft
created: 2026-10-18
source: ""
tags:
  - ollama
  - vision-language-models
  - qwen3-vl
  - image-analysis
  - comfyui
  - local-ai
  - msty
purpose: knowledge
---
## Overview

This article documents the selection of a local vision-language model (VLM) and lightweight desktop interface for analysing images using an NVIDIA RTX 5070 Ti.

The primary objective is to provide a local image-analysis environment capable of examining generated images and converting their visual characteristics into structured descriptions, natural-language image-generation prompts and Danbooru-style tags.

The proposed architecture separates image analysis from image generation:

- **Ollama:** Local model inference.
    
- **Qwen3-VL 8B:** Vision-language model for image understanding.
    
- **Msty:** Desktop interface for submitting images and prompts.
    
- **ComfyUI:** Independent image-generation environment.
    

This configuration was recommended during the original discussion. The complete Ollama–Qwen3-VL–Msty integration was not confirmed as tested.

## 1. Hardware and Software Environment

### Hardware

|Component|Specification|
|---|---|
|CPU|AMD Ryzen 7 9800X3D|
|GPU|Gigabyte RTX 5070 Ti Windforce OC SFF|
|VRAM|16 GB|
|RAM|32 GB DDR5|
|Storage|Corsair MP600 Pro LPX 2 TB NVMe|
|Operating system|Windows 11 Home|

The system has successfully completed CPU, GPU and storage validation, including zero reported PCIe and WHEA errors during testing.

### Software baseline

- Ollama installed.
    
- ComfyUI Desktop installed and operational.
    
- CUDA-based image generation verified.
    
- Stable Diffusion WebUI Forge previously tested but no longer the preferred image-generation environment.
    
- Ollama configured for deliberate startup rather than unnecessary background GPU consumption.
    

ComfyUI's shared model directory is:

```
C:\Users\darre\AppData\Local\Comfy-Desktop\ComfyUI-Shared\models
```

This directory belongs to the image-generation environment and is not assumed to be Ollama's model storage location.

## 2. Image Analysis Requirements

The intended use is substantially more sophisticated than basic image classification.

The vision model should be able to analyse:

- **Subjects:** Physical appearance, facial characteristics, hairstyle, clothing and accessories.
    
- **Pose:** Body position, orientation, gestures and interactions.
    
- **Expression:** Facial expression, gaze direction and visible emotion.
    
- **Composition:** Framing, subject placement, perspective, depth and camera angle.
    
- **Lighting:** Direction, intensity, shadows, colour temperature and atmospheric effects.
    
- **Style:** Photographic, illustrative, anime and other visual characteristics.
    
- **Environment:** Background objects, architecture, scenery and spatial relationships.
    

The model should then be capable of translating its observations into image-generation terminology.

Two important output formats are:

1. Natural-language prompts suitable for Krea 2 and similar models.
    
2. Danbooru-style tags suitable for Illustrious-derived checkpoints.
    

These formats should be treated separately because different image-generation models interpret prompting conventions differently.

## 3. Recommended Vision Models

### Primary recommendation: Qwen3-VL 8B

**Model:** `qwen3-vl:8b`

**Source:** [https://ollama.com/library/qwen3-vl](https://ollama.com/library/qwen3-vl)

Qwen3-VL 8B was selected as the initial candidate because it combines multimodal image understanding with a model size appropriate for the available GPU.

Relevant capabilities include:

- Image understanding and description.
    
- Spatial reasoning.
    
- Object and attribute recognition.
    
- Text recognition within images.
    
- Detailed visual descriptions.
    
- Conversion of image observations into structured text.
    

The Ollama model package was reported as approximately 6.1 GB during the original evaluation.

This is a model storage size, not a guarantee of runtime VRAM consumption. Actual memory requirements also depend on context length, image processing, quantisation and Ollama's runtime behaviour.

#### Initial installation and testing

```
ollama pull
```