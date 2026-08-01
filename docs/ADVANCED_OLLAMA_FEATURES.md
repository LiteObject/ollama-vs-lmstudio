---
title: "Advanced Ollama Features and Configuration Guide 2026"
description: "Learn advanced Ollama features including custom Modelfiles, API integration, GPU optimization, Docker deployment, and production configuration."
keywords: "Ollama advanced features, Ollama API, Ollama Docker, Ollama GPU optimization, Ollama Modelfiles, Ollama production setup"
---

# Advanced Ollama Features and Configuration Guide 2026

*Okay, so you've got the basics down and want to see what else Ollama can do. Here's the fun stuff.*

Learn advanced Ollama features including custom Modelfiles, API integration, GPU optimization, Docker deployment, and production configuration for local AI models.

> Everything here is checked against the [official Ollama docs](https://docs.ollama.com/). If a flag or variable ever looks wrong, `ollama serve --help` is the source of truth for server settings.

## Creating custom models with Modelfiles

Think of Modelfiles like recipes - you take an existing model and customize how it behaves:

```dockerfile
FROM qwen3.5:9b
SYSTEM """You are a Python coding expert who explains things clearly."""
PARAMETER temperature 0.1
PARAMETER num_ctx 8192
```

```bash
ollama create python-expert -f ./Modelfile
ollama run python-expert
```

Now you've got a model that's specifically tuned to help with Python and gives more focused, less random responses.

You usually **don't** need a `TEMPLATE` line - Ollama inherits the base model's chat template, and getting it wrong is the fastest way to break a model. Only override it if you know exactly what the model expects.

## Running multiple models at once

The Ollama server handles this for you - there's no need to background a process per model. If a model fits in available memory, it gets loaded alongside the others:

```bash
# See what models you have
ollama list

# See what's actually loaded right now, and where
ollama ps

# Free memory when you're done with one
ollama stop qwen3.5:9b
```

By default Ollama will keep up to 3 models loaded (`OLLAMA_MAX_LOADED_MODELS`), unloading idle ones when it needs room. Just hit the API with a different `model` value and the right one gets loaded.

*Side note:* on Linux and macOS, a trailing `&` (as in `ollama serve &`) runs a command in the background so it doesn't block your terminal. That's useful for the server itself, but don't background a bunch of `ollama run` commands to "load" several models - `ollama run` is an interactive chat session, not a way to preload.

### Switching between models via API
```python
import requests

def ask_model(model, question):
    response = requests.post('http://localhost:11434/api/generate',
        json={'model': model, 'prompt': question, 'stream': False})
    response.raise_for_status()
    return response.json()['response']

# Use different models for different things
code_answer = ask_model('qwen3-coder:30b', 'Write a Python function to sort a list')
general_answer = ask_model('qwen3.5:9b', 'Explain quantum computing simply')
reasoning_answer = ask_model('gpt-oss:20b', 'Solve this complex logic puzzle: ...')
```

Pretty neat - you can have a coding specialist and a general knowledge model available side by side.

## GPU optimization (making things faster)

Ollama is pretty smart about using your GPU automatically, but you can tweak things:

### Automatic GPU detection
Ollama automatically finds and uses your GPU (NVIDIA, AMD, or Apple's chips). Usually it just works.

### Manual tweaking
```bash
# Pick a specific GPU when you have several
CUDA_VISIBLE_DEVICES=0 ollama serve      # NVIDIA
ROCR_VISIBLE_DEVICES=0 ollama serve      # AMD

# Cut memory use as context grows (on by default where supported)
OLLAMA_FLASH_ATTENTION=1 ollama serve

# Shrink the K/V cache: f16 (default), q8_0, or q4_0
OLLAMA_KV_CACHE_TYPE=q8_0 ollama serve

# Change the default context window (default is 4096 tokens)
OLLAMA_CONTEXT_LENGTH=8192 ollama serve
```

To control how many layers land on the GPU, use the `num_gpu` **model parameter** rather than an environment variable:

```bash
# Inside an interactive session
/set parameter num_gpu 35
```

```json
// Or per-request via the API
{ "model": "qwen3.5:9b", "prompt": "hi", "options": { "num_gpu": 35 } }
```

### Check what's happening
```bash
# See GPU usage (NVIDIA cards)
nvidia-smi

# See what Ollama is doing
ollama ps  # Shows loaded models and whether they're on GPU or CPU
```

The `PROCESSOR` column in `ollama ps` is the number that matters: `100% GPU` is what you want, `100% CPU` means the model didn't fit.

**Reality check:** The defaults usually work fine. Only mess with this if you're having performance issues.

## Model quantization levels (quality vs size trade-offs)

Different compression levels of the same model - think video quality settings. Check the model's tag list on [ollama.com/library](https://ollama.com/library) for what's actually published:

```bash
# Different quality levels of the same model
ollama pull gemma4:12b-it-qat     # Quantization-aware trained - best quality per GB
ollama pull gemma4:12b-it-q4_K_M  # Good balance (this is what the plain tag gives you)
ollama pull gemma4:12b-it-q8_0    # Larger file, higher quality
ollama pull gemma4:12b-it-bf16    # Essentially uncompressed, very large
```

On Apple Silicon, look for `-mlx` tags. On recent NVIDIA hardware, `-nvfp4` and `-mxfp8` are worth a try.

### Custom quantization in Modelfiles
```dockerfile
# Modelfile pinned to a specific quantization
FROM gemma4:12b-it-q4_K_M
SYSTEM "You are a helpful assistant."
PARAMETER num_ctx 8192  # How much context to remember
```

**Practical advice:** q4_K_M is the sweet spot for most people, and a `qat` build is better still when one exists. Only go higher if you have tons of RAM and want max quality.

## API integration for developers

This is where Ollama really shines - you can integrate it into your own apps:

### Basic REST API usage

**Simple example - just copy and paste this!**

```python
# Easy version for beginners
import requests

# Ask the AI a question
response = requests.post('http://localhost:11434/api/generate',
    json={
        'model': 'qwen3.5:4b',
        'prompt': 'Tell me a joke about computers',
        'stream': False
    })

# Print what it says
print(response.json()['response'])
```

**More advanced version for developers:**

```python
# Chat completion using the official client
from ollama import chat

response = chat(
    model='qwen3.5:9b',
    messages=[{'role': 'user', 'content': 'Hello there!'}],
)
print(response.message.content)
```

```bash
pip install ollama    # Python
npm install ollama    # JavaScript / TypeScript
```

### OpenAI-compatible endpoint

If you already have code written against the OpenAI SDK, you barely have to change anything:

```python
from openai import OpenAI

client = OpenAI(base_url='http://localhost:11434/v1', api_key='ollama')  # key is ignored

response = client.chat.completions.create(
    model='qwen3.5:9b',
    messages=[{'role': 'user', 'content': 'Hello there!'}],
)
print(response.choices[0].message.content)
```

This is the easiest way to point an existing tool at your local models.

### LangChain integration (for RAG and complex workflows)
```python
from langchain_ollama import ChatOllama
from langchain.chains import RetrievalQA

# Connect Ollama to LangChain
llm = ChatOllama(model="qwen3.5:9b")

# Create a retrieval-augmented generation (RAG) pipeline
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=your_vector_store.as_retriever()
)
```

(The old `langchain_community.llms.Ollama` class is deprecated - use the `langchain-ollama` package.)

### Running in Docker
```bash
# Start the server with a persistent volume for models
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

# Pull a model into the running container
docker exec -it ollama ollama pull qwen3.5:9b

# On Linux/WSL2 with an NVIDIA GPU, add --gpus=all (needs nvidia-container-toolkit)
docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

Don't try to `RUN ollama pull` inside a Dockerfile - the pull needs a running server, so it has to happen at runtime.

### Plugging into your editor

```bash
ollama launch            # pick an integration interactively
ollama launch claude --model qwen3-coder:30b
```

This configures supported coding tools (VS Code, Claude Code, Codex, OpenCode, Droid) to talk to your local models.

This stuff gets pretty technical, but it's powerful once you get the hang of it.

## Fine-tuning behavior with parameters

There are no `--temperature` style flags on `ollama run`. You change sampling either inside the session, in a Modelfile, or per API request:

```bash
# Inside an interactive session
>>> /set parameter temperature 0.9
>>> /set parameter top_p 0.9
>>> /set parameter num_ctx 8192
>>> /show parameters
```

```python
# Or per request through the API
requests.post('http://localhost:11434/api/chat', json={
    'model': 'qwen3.5:9b',
    'messages': [{'role': 'user', 'content': 'Write me a limerick'}],
    'options': {'temperature': 0.9, 'top_p': 0.9, 'repeat_penalty': 1.1},
    'stream': False,
})
```

Lower temperature (0.1-0.3) for focused, factual work. Higher (0.8-1.0) for creative writing.

### Environment variables for server configuration

These configure the **server**, so set them before `ollama serve` (or in your systemd unit / Windows env vars - see the [Ollama FAQ](https://docs.ollama.com/faq)):

```bash
# Make Ollama accessible from other computers on your network
export OLLAMA_HOST=0.0.0.0:11434      

# Allow requests from specific web origins
export OLLAMA_ORIGINS="https://myapp.com"

# Store models somewhere else
export OLLAMA_MODELS="/custom/path"    

# Handle more simultaneous requests per model (default 1)
export OLLAMA_NUM_PARALLEL=4           

# Keep more models loaded in memory (default 3)
export OLLAMA_MAX_LOADED_MODELS=3      

# How long an idle model stays loaded (default 5m)
export OLLAMA_KEEP_ALIVE=30m

# Default context window in tokens (default 4096)
export OLLAMA_CONTEXT_LENGTH=8192

# Queue depth before the server returns 503 (default 512)
export OLLAMA_MAX_QUEUE=512

# Turn off cloud models and web search entirely
export OLLAMA_NO_CLOUD=1
```

Note that `OLLAMA_NUM_PARALLEL` multiplies memory use: required RAM scales with `OLLAMA_NUM_PARALLEL` × `OLLAMA_CONTEXT_LENGTH`.

### Advanced Modelfile example

**Note:** Most of the time you only need `FROM`, `SYSTEM`, and a couple of `PARAMETER` lines. Everything else is optional.

```dockerfile
FROM qwen3.5:9b

SYSTEM """You are a helpful AI assistant with a sense of humor."""

PARAMETER temperature 0.7
PARAMETER num_ctx 8192

# Seed the model with example turns so it copies the style
MESSAGE user What's the deal with airline food?
MESSAGE assistant Honestly? It's the only meal where the altitude is higher than the expectations.

# Refuse to load on an Ollama version that's too old for this file
REQUIRES 0.14.0
```

**Honestly:** The defaults work fine for most people. Only mess with this stuff if you have specific needs.

## Managing your model collection

### Versioning and tagging
```bash
# Create different versions of customized models
ollama create my-assistant:v1 -f ./Modelfile.v1
ollama create my-assistant:v2 -f ./Modelfile.v2
ollama create my-assistant:latest -f ./Modelfile.latest

# See all your models and versions
ollama list | grep my-assistant

# Clean up old versions
ollama rm my-assistant:v1
```

### Backup and sharing
```bash
# Export a model's configuration for backup
ollama show --modelfile my-assistant > my-assistant.Modelfile

# Import on another computer
ollama create my-assistant -f ./my-assistant.Modelfile

# Copy models between different names
ollama cp source-model target-model
```

This is handy when you've spent time tweaking a model and want to save that configuration. Note that the exported Modelfile references a local blob path, so edit the `FROM` line back to the original model name before using it elsewhere.

## Production deployment (if you're building something serious)

### Load balancing with Docker Compose
```yaml
# docker-compose.yml for running multiple Ollama instances
services:
  ollama-1:
    image: ollama/ollama
    ports: ["11434:11434"]
    volumes: ["ollama-1:/root/.ollama"]
  
  ollama-2:
    image: ollama/ollama
    ports: ["11435:11434"]
    volumes: ["ollama-2:/root/.ollama"]
  
  nginx:
    image: nginx
    ports: ["80:80"]
    volumes: ["./nginx.conf:/etc/nginx/nginx.conf:ro"]

volumes:
  ollama-1:
  ollama-2:
```

Give each instance its own volume - two servers writing to one model directory will fight over it.

### Monitoring and logging
```bash
# Debug mode for troubleshooting
OLLAMA_DEBUG=1 ollama serve

# System service monitoring (Linux)
sudo systemctl status ollama
sudo journalctl -u ollama -f

# Health check
curl http://localhost:11434/api/tags
```

### Security for production

**Ollama has no built-in authentication.** Anything that can reach port 11434 can run models, pull new ones, and delete yours. Treat it like an unauthenticated internal service.

```bash
# Default and safest: listen on localhost only
export OLLAMA_HOST=127.0.0.1:11434  

# Only allow specific web origins (avoid "*" on a shared network)
export OLLAMA_ORIGINS="https://myapp.com"  
```

If it needs to be reachable by anything else:
- Put a reverse proxy (nginx, Caddy, Traefik) in front of it and terminate TLS there
- Add authentication at the proxy - API keys, mTLS, or SSO
- Restrict source addresses with a firewall or security group
- Never expose port 11434 directly to the internet

## Performance optimization tricks

### Memory management

Ollama sizes things automatically based on available VRAM and system RAM. The knobs that actually exist:

```bash
export OLLAMA_KV_CACHE_TYPE=q8_0   # Roughly halves K/V cache memory
export OLLAMA_FLASH_ATTENTION=1    # Lower memory growth at long context
export OLLAMA_CONTEXT_LENGTH=4096  # Smaller context = less memory
export OLLAMA_MAX_LOADED_MODELS=1  # Don't hold several models at once
```

If a model won't fit, the real fixes are a smaller model, a smaller quantization, or a shorter context - not a magic variable.

### Batch processing for efficiency
```python
# Process multiple requests efficiently
import asyncio
import aiohttp

async def process_multiple_prompts(prompts, model="qwen3.5:9b"):
    async with aiohttp.ClientSession() as session:
        async def one(prompt):
            async with session.post('http://localhost:11434/api/generate',
                    json={'model': model, 'prompt': prompt, 'stream': False}) as r:
                return await r.json()

        return await asyncio.gather(*(one(p) for p in prompts))
```

Set `OLLAMA_NUM_PARALLEL` above 1 or these will just queue up one at a time.

### Smart caching
```bash
# Keep models loaded longer (saves startup time)
export OLLAMA_KEEP_ALIVE=24h         

# Cache multiple models in memory
export OLLAMA_MAX_LOADED_MODELS=5    

# Preload a model you use frequently
ollama run qwen3.5:9b ""
```

You can also set `keep_alive` per request - `-1` pins a model in memory, `0` unloads it as soon as it responds.

## Some practical advice from experience

### Development workflow that actually works
1. **Start simple** - Get basic functionality working first
2. **Iterate quickly** - Use Modelfiles to test different behaviors rapidly
3. **Monitor everything** - Keep an eye on resource usage, especially at first
4. **Version your models** - Tag different configurations so you can roll back
5. **Test thoroughly** - AI models can be unpredictable, so test edge cases

### Production deployment checklist
- [ ] Set appropriate resource limits
- [ ] Configure monitoring and alerting
- [ ] Set up health checks
- [ ] Implement proper security (Ollama has no auth of its own - put a proxy in front of it)
- [ ] Keep `OLLAMA_HOST` bound to localhost unless you deliberately need otherwise
- [ ] Decide whether cloud models are acceptable, and set `OLLAMA_NO_CLOUD=1` if not
- [ ] Plan for model updates and rollbacks
- [ ] Document your API usage and parameters

## Working with vision models (text + images)

### Getting started with vision models
Most current models understand both text and images. This is genuinely useful:

```bash
# Download a vision model
ollama pull qwen3-vl:8b

# Pass the image by putting its path in the prompt
ollama run qwen3-vl:8b "What's in this image? ./photo.jpg"
```

There's no `--image` flag - Ollama picks the path out of the prompt text.

### API usage with images
```python
import base64
import requests

def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')

def ask_about_image(image_path, question):
    base64_image = encode_image(image_path)
    
    response = requests.post('http://localhost:11434/api/generate',
        json={
            'model': 'qwen3-vl:8b',
            'prompt': question,
            'images': [base64_image],
            'stream': False
        })
    
    return response.json()['response']

# Example usage
result = ask_about_image("screenshot.png", "Describe what you see in this screenshot")
print(result)
```

### Practical vision model tips
- **qwen3-vl:8b** - Best balance of capability and resource usage
- **gemma4:12b** - Vision built into a strong general-purpose model
- **minicpm-v4.5:8b** - Strong at images and video frames
- **glm-ocr** / **deepseek-ocr** - Purpose-built for document text extraction

These models can:
- Describe images and photos
- Read text from screenshots
- Answer questions about charts and graphs
- Help with visual troubleshooting
- Analyze documents and diagrams

**Reality check:** Vision models need more resources than text-only models. Start around 8B unless you have plenty of RAM and GPU memory.

## Newer model capabilities

### Function calling and tool use
Most current models have built-in function calling. Look for the `tools` badge on the model's library page:

```bash
ollama run gpt-oss:20b        # OpenAI's open-weight models
ollama run qwen3.5:9b         # Multimodal, tools, and thinking
ollama run granite4.1:8b      # IBM, strong at structured output
ollama run lfm2.5:8b          # Built specifically for fast tool calling
```

### Configurable reasoning effort
Models that support thinking expose it through the `think` parameter, not a sampling option. gpt-oss accepts an effort level:

```python
import requests

def ask_with_reasoning(question, effort="medium"):
    response = requests.post('http://localhost:11434/api/chat',
        json={
            'model': 'gpt-oss:20b',
            'messages': [{'role': 'user', 'content': question}],
            'think': effort,   # low, medium, high
            'stream': False
        })
    return response.json()['message']['content']

# Example usage
quick_answer = ask_with_reasoning("What's 2+2?", "low")
complex_answer = ask_with_reasoning("Solve this complex logic puzzle...", "high")
```

For models that just toggle thinking on or off, pass `'think': True` or `False`. From the CLI, use `--think` and `--hidethinking`.

### Multilingual coding
The Qwen coding models handle prompts in many languages, which is handy on international teams:

```bash
ollama run qwen3-coder:30b "Write a Python function to sort a list"
ollama run qwen3-coder:30b "写一个Python函数来排序列表"
```

### Advanced mathematical reasoning
Reasoning models are the ones to reach for on proofs and hard math:

```bash
ollama run gpt-oss:20b "Prove that the square root of 2 is irrational"
```

These newer models represent significant advances in local AI capabilities, bringing features that were previously only available in cloud services.

**Bottom line:** Ollama is surprisingly powerful once you dig into it. You can build some pretty sophisticated AI applications while keeping everything running on your own hardware. The learning curve isn't too steep, and the flexibility is worth it.