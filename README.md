---
title: "How to Run AI Models Locally - Free ChatGPT Alternative 2026"
description: "Learn to install and run AI models like Qwen 3.5, Gemma 4, and gpt-oss on your computer for free. Step-by-step guide to local AI with LM Studio and Ollama."
keywords: "local AI, run AI models locally, free ChatGPT alternative, Ollama, LM Studio, Qwen 3.5, Gemma 4, gpt-oss, local AI installation"
---

# How to Run AI Models Locally on Your Computer - Free ChatGPT Alternative

Tired of paying a monthly subscription for a chatbot? Yeah, me too. Here's how to run these AI models on your own machine - completely free and private. Learn to install and use local AI models like Qwen 3.5, Gemma 4, and gpt-oss with step-by-step instructions.

**Complete documentation available in the [docs folder](./docs/)**

## Benefits of Running AI Models Locally vs Cloud Services

I was skeptical at first. But after using local AI for months, I'm never going back to paid services for most tasks. Here's why:

- **It's actually free** - No subscription fees eating into your budget
- **Your conversations stay private** - Nothing gets sent to some company's servers
- **Works when your internet doesn't** - Perfect for flights or sketchy WiFi
- **You can tinker with it** - Want to modify how the AI behaves? Go for it.

## Step 1: Choose the Best Local AI Software (LM Studio vs Ollama)

Both tools now ship a desktop app *and* a command line, so the old "GUI vs terminal" split isn't as sharp as it used to be. Pick either one - you can't really go wrong.

### If you hate command lines
**[LM Studio](https://lmstudio.ai/)** - This one's got the nicest interface
1. Download it and install (pretty straightforward)
2. Browse models in the app - they've got tons
3. Hit download, wait a bit, then start chatting

### If you're okay with typing commands
**[Ollama](https://ollama.com/)** - My personal favorite
```bash
# Mac/Linux folks:
curl -fsSL https://ollama.com/install.sh | sh

# Windows people: Just download the installer from the website

# Then try this:
ollama run qwen3.5:4b
```

Ollama also has a desktop app for Windows, macOS, and Linux now, so you get a chat window out of the box - the terminal is optional.

**Heads up on cloud models:** Ollama's library includes "cloud" tagged models that run on Ollama's servers, not your machine. Handy, but they aren't private and they aren't local. Stick to regular tags if privacy is the point. You can turn the feature off entirely with `OLLAMA_NO_CLOUD=1`.

## Step 2: Check Your Computer Hardware Requirements for AI Models

This is important - you can't run huge models on a slow computer. But don't worry, there are good options for everyone.

### VRAM is what matters, not RAM

This trips up almost everyone, so let's be blunt about it:

- **If the whole model fits in your GPU's VRAM**, it's fast - dozens of words per second.
- **If it doesn't**, the leftover layers run on your CPU out of system RAM, which is typically 5-20x slower. It still works, it's just painful.
- **Apple Silicon is the exception.** RAM and VRAM are one shared pool, though macOS reserves some for the system - budget for roughly 70% of your total.

So "do I have 32GB of RAM?" is a much less useful question than "do I have 12GB of VRAM?"

### What actually fits

Sizes below are for the default Q4_K_M quantization - what you get when you don't specify a tag.

| Your VRAM (or unified memory) | Look for downloads up to | Example that fits |
|---|---|---|
| No GPU, 8GB system RAM | ~4GB | `qwen3.5:4b` (3.4GB) or `granite4.1:3b` (2.1GB) |
| 8GB | ~6GB | `granite4.1:8b` (5.3GB) |
| 12GB | ~10GB | `gemma4:12b` (7.6GB) or `qwen3.5:9b` (6.6GB) |
| 16GB | ~14GB | `gpt-oss:20b` (14GB) |
| 24GB | ~21GB | `qwen3.5:27b` (17GB) or `granite4.1:30b` (17GB) |
| 32GB+ VRAM, or 64GB+ unified | ~29GB and up | `qwen3.5:35b` (24GB) |

Rule of thumb: **the download size is roughly the memory the model needs**, plus overhead for context. At the default 4K context that overhead is small; at 32K and beyond it can add several GB, so leave headroom.

### When it doesn't fit

Ollama runs the model anyway, splitting it between GPU and CPU. Check what actually happened:

```bash
ollama ps
```

The `PROCESSOR` column tells you everything:
- `100% GPU` - ideal, full speed
- `48%/52% CPU/GPU` - partial offload, noticeably slower but usable
- `100% CPU` - no GPU acceleration at all, expect a crawl

Stuck on partial offload? Drop a size (9B to 4B), shrink the context (`OLLAMA_CONTEXT_LENGTH=4096`), or shrink the KV cache (`OLLAMA_KV_CACHE_TYPE=q8_0`).

### Will my GPU even work?

- **NVIDIA** - Compute capability 5.0+ and driver 550 or newer. That's GTX 750 Ti and up, so most cards from the last decade. Compute capability 5.0-6.2 needs driver 570+. [Check your card](https://developer.nvidia.com/cuda-gpus)
- **Apple Silicon** - Every M-series chip works via Metal, no setup required
- **AMD on Linux** - Needs the ROCm v7 driver. RX 9000 and 7000 series, RX 6800 and above, Radeon PRO W6000/W7000, Ryzen AI, and Instinct cards
- **AMD on Windows** - Shorter list: RX 7900 XTX/XT/GRE, 7800 XT, 7700 XT, 7600 and 7600 XT, plus PRO W7000 series
- **Intel and others** - Covered by Vulkan on Windows and Linux, enabled by default

No supported GPU? Everything still runs on CPU - just stay at 4B and under.

**Not sure what you have?** 
- Windows: Right-click "This PC" → Properties for RAM; Task Manager → Performance → GPU for VRAM
- Mac: Apple Menu → About This Mac
- Linux: `lscpu` and `free -h`, plus `nvidia-smi` or `rocminfo` for the GPU

## Step 3: Download and Install Your First AI Model

**TL;DR:** Start with qwen3.5:4b - it's like the Honda Civic of AI models: reliable, efficient, and works for most people.

I've tried a bunch of these local AI models. Here are the best free ChatGPT alternatives that actually work well:

- **qwen3.5:4b** (~3.4GB) - Start here. Fast, runs on almost anything, handles text and images, and it's current
- **gemma3:4b** (~3.3GB) - Google's small model. Rock solid for everyday tasks
- **granite4.1:3b** - IBM's small model, good at following instructions and calling tools
- **qwen3.5:9b** (~6.6GB) - Noticeably smarter if you've got 16GB of RAM to spare
- **qwen3.5:2b** (~2.7GB) - When your machine is really tight on memory
- **gpt-oss:20b** - OpenAI's open-weight model, if you've got the hardware for it

Honestly, just start with `qwen3.5:4b`. You can always download more later (and trust me, you will).

**[Detailed model breakdown](docs/MODEL_GUIDE.md)**
**[What I'm using right now](docs/CURRENT_MODEL_RECOMMENDATIONS.md)**

## Step 4: How to Start Using Local AI Models

### If you went with LM Studio
1. Download a model from the search tab (I'd suggest Qwen3.5 4B)
2. Switch to the chat tab
3. Pick your model from the dropdown and start typing

### If you went with Ollama
```bash
ollama run qwen3.5:4b
>>> Hey there! What can I help you with?
```

A few commands worth knowing:

```bash
ollama list          # what you've downloaded
ollama ps            # what's loaded in memory right now
ollama stop <model>  # free up the memory
ollama rm <model>    # delete a model you're done with
```

That's it. You're now running AI on your own machine. Pretty cool, right?

## Troubleshooting Common Local AI Installation Issues

**Model running like molasses?** Try something smaller like `qwen3.5:2b` or `gemma3:1b`. Run `ollama ps` - if the `PROCESSOR` column says `100% CPU`, the model didn't fit in your GPU and that's your answer.

**Computer says "out of memory"?** Your machine needs more RAM, or switch to a smaller model (try going from 9B to 4B). Shrinking the context window helps too: `/set parameter num_ctx 4096` inside `ollama run`.

**Model "thinks" forever before answering?** You grabbed a reasoning model. That's normal behavior for them - pick a non-thinking model if you just want quick answers.

**Installation failing?** Restart your computer and check if your antivirus is being overly paranoid - sometimes it blocks AI software.

## Additional Local AI Resources and Guides

### Complete Documentation Index

#### Getting Started
- [**What Are AI Models?**](./docs/WHAT_ARE_AI_MODELS.md) - If you're curious how this magic works
- [**Tool Comparison**](./docs/TOOL_COMPARISON.md) - Deep dive into your options
- [**Ollama vs LM Studio**](./docs/Ollama_vs_LM_Studio.md) - Head-to-head on the two most popular tools

#### Model Selection and Recommendations  
- [**Model Guide**](./docs/MODEL_GUIDE.md) - Which models are actually good
- [**Current Favorites**](./docs/CURRENT_MODEL_RECOMMENDATIONS.md) - What I'm using lately

#### Technical Details
- [**File Formats Explained**](./docs/MODEL_FORMATS_AND_TYPES.md) - The technical stuff
- [**Advanced Ollama Tricks**](./docs/ADVANCED_OLLAMA_FEATURES.md) - For when you want to get fancy

### Quick Navigation

**New to AI?** Start with [What Are AI Models?](./docs/WHAT_ARE_AI_MODELS.md)

**Need model recommendations?** Check [Current Model Recommendations](./docs/CURRENT_MODEL_RECOMMENDATIONS.md)

**Want advanced features?** See [Advanced Ollama Features](./docs/ADVANCED_OLLAMA_FEATURES.md)

## Quick Help for Local AI Setup Issues

- 90% of problems are solved by trying a smaller model first
- Check if your antivirus is blocking stuff
- Don't expose Ollama's port (11434) to the internet - it has no authentication of its own
- When in doubt, restart and try again