---
date: '2026-01-18T13:20:00+09:00'
title: 'Free Ride on Apple Intelligence: Turn macOS Built-in Model into an OpenAI API'
slug: 'apple-foundation-model-local-api'
categories: ['AI']
tags: ['Apple Intelligence', 'macOS', 'LLM', 'Local AI', 'OpenAI API']
---

macOS 26 Tahoe ships with a 3B parameter language model that Apple calls the Foundation Model. It normally hides in the system powering Writing Tools, Siri, and other features, but you can actually pull it out, wrap it as an OpenAI-compatible API, and plug it into any tool that supports the OpenAI format.

Runs completely locally, no network calls, no cost.

## Prerequisites

You need macOS 26 Tahoe (now at 26.3 stable) and an Apple Silicon Mac. Intel Macs won't work because Apple Intelligence only supports M-series chips.

## Installation

There's an open-source project called maclocal-api that does something simple: it calls Apple's FoundationModels framework and spins up a local HTTP server exposing standard endpoints like `/v1/chat/completions`.

```bash
git clone https://github.com/scouzi1966/maclocal-api
cd maclocal-api
swift build -c release
```

Compilation takes a bit because it depends on the Vapor framework and pulls down a bunch of Swift packages. Took about two minutes on my machine.

Once compiled, the binary is at `.build/release/afm`.

## Starting the Server

```bash
./.build/release/afm --port 9999 --hostname 0.0.0.0
```

You'll see a fancy ASCII art logo, then the server is up. Default port is 9999.

For background running:

```bash
nohup ./.build/release/afm --port 9999 --hostname 0.0.0.0 > /tmp/afm.log 2>&1 &
```

## Usage

The API is fully OpenAI-compatible, so you can use the OpenAI SDK directly:

```python
from openai import OpenAI

client = OpenAI(
    api_key="not-needed",
    base_url="http://localhost:9999/v1"
)

response = client.chat.completions.create(
    model="foundation",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

Or curl:

```bash
curl http://localhost:9999/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "foundation", "messages": [{"role": "user", "content": "Hello"}]}'
```

Streaming is supported—just add `"stream": true`.

## Real-World Testing

I ran a bunch of tests. Conclusion: it works, but don't expect too much.

**What it can do:**

Translation works fine. Asked it to translate "今天天气真好" to English, got "Today, the weather is really good." Pretty natural.

Code generation is decent. Asked for a Fibonacci function, got clean Python with comments and usage examples.

Simple reasoning works. "If Alice is 3 years older than Bob, and Bob is 10, how old is Alice?" Answer: "13." Correct.

System prompts are followed. Told it to act like a cat, it actually replied "meow meow meow meow."

Summarization and rewriting work well—these are what Writing Tools already uses under the hood.

**What it can't do:**

No function calling / tool use. This is a pure text model, can't invoke tools.

Complex reasoning fails. With only 3B parameters, it can't compete with Claude or GPT-4.

No image generation. Apple's image generation is a separate thing (Image Playground), not exposed through this API.

Vision is fake. maclocal-api has an `afm vision` command, but it's just Apple's OCR framework extracting text, not real multimodal understanding.

## Resource Usage

This is the pleasant surprise.

The afm server itself uses only 33MB of RAM, 0% CPU at idle, around 0.7% during requests.

Why so light? Because the model inference isn't done by this process. Apple's Foundation Model is hosted by the system service `IntelligencePlatformComputeService`, which is already running in the background (for Writing Tools), so launching afm adds almost zero overhead.

In other words, the model is already loaded. afm is just a thin API wrapper.

## Use Cases

What is this good for?

Local privacy scenarios. Data never leaves your machine. If you're handling sensitive content, this is safer than any cloud API.

Free alternative for simple tasks. Translation, summarization, simple Q&A—this is enough, save some API costs.

Offline environments. On a plane, no internet? Still works.

What is it not good for?

Complex AI agent workflows. No tool calling means no complex orchestration.

High-quality output requirements. A 3B model has its ceiling. Don't expect stunning copy.

## How It Differs from Ollama

You might ask, how is this different from Ollama?

Ollama requires you to download models yourself, taking up several GB to tens of GB of disk space. Apple's Foundation Model comes with the system, no extra storage needed.

With Ollama you can swap models—want Llama 3? Run Llama 3. Want Qwen? Run Qwen. With Apple, you only get one 3B foundation model, no choice.

Performance-wise, Apple's model is optimized for their chips, inference is fast. But the 3B parameter ceiling is what it is. If you run a 70B model on Ollama, it'll definitely perform better.

So these aren't replacements for each other, more like complements. Light tasks use Apple's, heavy lifting use Ollama with big models.

## Wrap Up

Apple buried a free local LLM in macOS 26. Limited capabilities, but zero cost, zero configuration, zero privacy concerns. If you're a Mac user, worth spending five minutes to set up. At minimum, you can use it for OCR and simple text processing.

GitHub: https://github.com/scouzi1966/maclocal-api
