---
title: "AI Model File Formats Guide: GGUF, MLX, Safetensors Explained 2026"
description: "Complete guide to AI model file formats including GGUF, MLX, safetensors, and quantization levels. Learn which formats work best for your computer."
keywords: "GGUF format, MLX models, safetensors, AI model quantization, model file formats, Q4_K_M explained, MXFP4, AI model compression"
---

# AI Model File Formats Guide: GGUF, MLX, Safetensors Explained (2026)

*Look, AI model files are like video formats - they all contain the same "movie" but packaged differently. Some work better on your phone, others on your computer. Let me break down what you actually need to know.*

Complete guide to AI model file formats including GGUF, MLX, safetensors, and quantization levels. Learn which formats work best for your computer and how to choose the right AI model files.

## What's with all these different file formats?

Just like you can save a photo as JPG, PNG, or HEIC, AI models can be saved in different formats. Each one has trade-offs between file size, quality, and what devices can actually run them.

The short version: **you probably want GGUF, and Q4_K_M is a safe default**. If you're on an Apple Silicon Mac, MLX builds are usually faster. Everything else is details.

## The "full quality" formats (probably not for you)

*These are like RAW photo files - perfect quality but massive*

| Format | What it is | When you'll see it |
|--------|------------|-------------------|
| `.safetensors` | The standard format for original weights | Downloading from Hugging Face, or serving with vLLM |
| `.pt` / `.pth` | PyTorch's format | Research models |
| `.bin` | Old binary format | Older models |
| `.ckpt` | Checkpoint format | Legacy AI models |

**Reality check:** A full-precision 70B model is well over 100GB and needs server-grade hardware. Skip these unless you're fine-tuning or running a serving stack like vLLM.

## Compressed formats (this is what you want)

*Like streaming video - smaller files with acceptable quality loss*

Here's the thing: uncompressed AI models are huge. So smart people figured out how to compress them while keeping most of the intelligence intact.

| Format | What it does | Best for |
|--------|--------------|----------|
| **GGUF** | The standard compression everyone uses | Your home computer, any OS |
| **MLX** | Apple's on-device format | Macs with M-series chips |
| **MXFP4 / NVFP4** | Newer 4-bit floating point formats | Recent NVIDIA GPUs, and how gpt-oss ships |
| **AWQ / GPTQ** | GPU-optimized 4-bit quantization | vLLM and similar serving stacks |
| **GGML** | GGUF's predecessor | Don't bother - GGUF replaced it years ago |

### GGUF compression levels (the important part)

*Think Netflix quality settings - higher = better picture but bigger download*

These examples assume a 7B-9B model. Bigger models scale up proportionally.

- **Q2_K** - Smallest, lowest quality (like 240p video - rough but functional)
  - **Example:** 7B model → ~3GB file
  - **Use when:** You're on an old laptop with 8GB RAM and want to save space
  
- **Q3_K_M** - Small, decent quality (480p - acceptable for many things)
  - **Example:** 7B model → ~4GB file  
  - **Use when:** You want decent quality but need to save disk space

- **Q4_K_M** - **← This is the sweet spot** (720p - great balance)
  - **Example:** 7B model → ~5GB file
  - **Use when:** You're not sure what to pick - this works for 90% of people
  
- **Q5_K_M** - Larger, high quality (1080p - really good)
  - **Example:** 7B model → ~6GB file
  - **Use when:** You have plenty of RAM and want better responses
  
- **Q6_K** - Large, very high quality (4K - excellent but big)
  - **Example:** 7B model → ~8GB file
  - **Use when:** You have 16GB+ RAM and want near-perfect quality
  
- **Q8_0** - Huge, near-perfect quality (like uncompressed - probably overkill)
  - **Example:** 7B model → ~10GB file
  - **Use when:** You're a perfectionist with tons of RAM

**My advice:** Always start with Q4_K_M. It's what most people use and it works great.

### The newer tags you'll bump into

- **`-qat`** - Quantization-aware training. The model was trained knowing it would be squeezed, so a QAT 4-bit build beats a normal 4-bit build at the same size. Grab these when offered
- **`-mxfp4` / `-nvfp4`** - 4-bit floating point instead of 4-bit integer. Better quality per byte on hardware that supports it
- **`-mlx`** - Apple Silicon builds. Noticeably faster on M-series Macs
- **`-bf16`** - Basically uncompressed. Big, and rarely worth it locally
- **`-cloud`** - Not a file at all. It runs on a remote server

## Specialized formats (probably skip this section)

*These are like apps made for specific phones - work great on one device, useless on others*

| Format | Made for | When you'd use it |
|--------|----------|-------------------|
| `.onnx` | Cross-platform compatibility | Windows ML, embedded, or NPU inference |
| `.tflite` / LiteRT | Mobile devices | Building a phone app |
| `.engine` / `.plan` (TensorRT) | NVIDIA GPUs only | Squeezing max performance from RTX cards |
| `.mlpackage` (Core ML) | Apple devices | iPhone/Mac app integration |
| OpenVINO IR | Intel CPUs, GPUs, and NPUs | Intel-specific optimization |

**Real talk:** Unless you're doing something very specific, stick with GGUF. These other formats are for specialized use cases.

## Types of AI models (what they actually do)

*Like hiring different types of writers for different jobs*

### The main categories

| Type | What it does | Like having... | Examples |
|------|--------------|----------------|----------|
| **Text generators** | Writes new content and answers questions | A capable assistant | Qwen 3.5, Gemma 4, gpt-oss |
| **Embedding models** | Turns text into vectors for search | A librarian who knows where everything is | EmbeddingGemma, Nomic Embed, Qwen3-Embedding |
| **Reranking / safety models** | Scores or filters other models' output | A quality control reviewer | Granite Guardian, ShieldGemma |

**For most people:** You want text generators. These are the ChatGPT-style models that can chat, write, and help with tasks. You'll only need an embedding model if you're building document search or RAG.

## Models organized by what they're actually good at

*Like hiring specialists for different jobs*

### The all-rounders (good at most things)
*Like a smart friend who can help with various tasks*

- **Qwen 3.5** - Alibaba's multimodal family, 0.8B up to 122B, 256K context throughout
- **Gemma 4** - Google's current family. Handles text, images, and audio
- **gpt-oss** - OpenAI's open-weight models (20B and 120B)
- **Granite 4.1** - IBM's Apache 2.0 family, strong on tool calling
- **Ministral 3** - Mistral's edge-focused models (3B/8B/14B)
- **LFM2.5** - Fast, hybrid-architecture models built for on-device use

### Programming helpers
*Like having a coding buddy who explains things well*

- **Qwen3-Coder** - Currently the best local coding assistant I've used
- **Devstral Small 2** - Purpose-built for agentic coding across many files
- **Qwen2.5-Coder** - Older, but still great on modest hardware
- **Granite 4.1** - Good for technical tasks and structured output

### Reasoning specialists
*Like having a smart friend who's great at working through complex problems*

- **gpt-oss** - Strong reasoning with adjustable effort levels
- **Qwen 3.5 / Qwen 3.6** - Thinking mode built into the same model
- **Olmo 3.1** - Fully open training data and recipes
- **DeepSeek R1** - The model that popularized open reasoning; still solid, but showing its age

### Vision models (can "see" images)
*Like having someone describe pictures to you*

- **Qwen3-VL** - Broad range of sizes, very capable
- **Gemma 4** - Vision (and audio) baked into the general-purpose model
- **MiniCPM-V 4.5 / 4.6** - Strong image and video understanding at small sizes
- **GLM-OCR / DeepSeek-OCR** - Document parsing and text extraction

**Bottom line:** Start with Qwen 3.5 4B, Gemma 3 4B, or Granite 4.1 3B. They're efficient and handle most tasks well. If you need reasoning help, try gpt-oss:20b.

## What format should you actually download?

*Stop overthinking it*

### If you have a regular computer (no gaming GPU)
- **GGUF Q4_K_M** - This is what most people use and it works great
- **GGUF Q5_K_M** - If you want slightly better quality and have extra RAM
- **A QAT build** - If one exists at your size, take it over the plain quant

### If you have an Apple Silicon Mac
- **MLX builds** - Faster than GGUF on M-series chips
- **GGUF** - Still works fine and is more portable

### If you have a gaming computer
- **GGUF** - Still the easy answer, and it uses your GPU
- **AWQ / MXFP4 / NVFP4** - If you're running vLLM or another serving stack

### If you're new to this
- **Always choose GGUF Q4_K_M** - It's compatible with everything and performs well
- Don't stress about the other formats until you have a specific reason to use them

## Simple rules for beginners

1. **Look for `.gguf` files** 
2. **Choose `Q4_K_M` compression** (best balance), or a `qat` build if offered
3. **Start with `2B` to `9B` models** (won't crash your computer)
4. **Pick `Qwen 3.5` or `Gemma 3`** for your first try

**Example of a good first choice:** `qwen3.5-4b-instruct-q4_k_m.gguf`

Translation: "Qwen 3.5, 4 billion parameters, instruction-tuned, Q4_K_M compression, GGUF format" - perfect for getting started!

If you're using Ollama, you never see the filename at all - `ollama run qwen3.5:4b` picks a Q4_K_M GGUF for you.