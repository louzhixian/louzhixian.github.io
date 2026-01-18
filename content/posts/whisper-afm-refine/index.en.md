---
date: '2026-01-18T17:15:00+09:00'
title: 'Add a Local “Editor” to Whisper: Cleaning Transcripts with Apple Foundation Model'
slug: 'whisper-afm-refine'
categories: ['AI']
tags: ['Apple Intelligence', 'macOS', 'Whisper', 'Local AI', 'Productivity']
cover:
  image: 'cover.png'
  alt: 'cover'
  relative: true
---

If you use voice-to-text often, you already know the problem: Whisper transcripts are usually not ready to ship. You get filler words, no punctuation, and sentences that run forever. You can either fix it manually, or you accept something that looks like a raw dump.

While testing Apple’s on-device Foundation Model, I found it’s surprisingly good at one specific job: acting like a tiny local editor. It turns a “spoken” transcript into something you can actually paste into notes, chat, or a doc.

## The Problem: Whisper Output Is Too Raw

A typical transcript looks like this (Chinese example):

> 今天打算做的事情还有哪些呢一会儿我觉得可以让AI助理帮我每天看一下我的Google Analytics里面我的Blog Site的访问情况毕竟我平时想不起来去看让他帮我追一下还有一件能做的事情就是就是什么来着怎么突然想不起来了那个算了先这样吧

No punctuation, no structure, and every filler phrase is preserved.

## The Idea: Whisper First, Then AFM Refine

Apple Foundation Model is small (3B), but for rewriting and cleanup it’s more than enough. I simply asked it to behave like an editor:

```
System Prompt: Rewrite a spoken transcript into written form. Remove filler words, remove duplicates, fix sentence boundaries, and add punctuation. Keep the meaning unchanged. Output only the cleaned text.
```

The same transcript becomes something like:

> 今天打算做的事情还有哪些呢？一会儿我想让AI助理每天查看我的Google Analytics，看看博客的访问情况。毕竟平时我常常忘记去看，让它帮我跟踪一下。还有一件事情，我突然想不起来了，算了先这样吧。

Readable, structured, and still faithful.

## Implementation: One HTTP API

I wrapped both Whisper and AFM into a single HTTP service. The request shape stays compatible with a typical Whisper `/asr` endpoint, but the returned text is already cleaned.

```python
@app.post("/asr")
async def asr(audio_file: UploadFile, refine: bool = True):
    raw_text = transcribe_audio(audio_file)  # mlx-whisper
    if refine:
        return await refine_with_afm(raw_text)  # Apple Foundation Model
    return raw_text
```

The core points are straightforward:

- Use `mlx-whisper` for fast local transcription on Apple Silicon
- Send the transcript to AFM via `/v1/chat/completions`
- Add a `refine` switch so you can opt out when needed
- Everything stays on-device (privacy-friendly)

## Performance: Under One Second Overhead

On my machine, refine adds about ~0.8 seconds.

| Mode | Time |
|------|------|
| Whisper only | ~1.05s |
| Whisper + AFM | ~1.85s |

That’s a very reasonable tradeoff for a transcript you can actually use.

## Deployment: Launchd on macOS

I run it as a launchd service (auto-start at boot) listening on port 9000. That way every transcription request automatically gets the cleanup pass, and callers don’t need to change how they consume the result.

## Where This Helps

This works especially well for:

- Voice notes
- Chat messages (voice → text → send)
- Meeting snippets
- Pre-cleaning subtitles before manual edits

## Limitations

AFM is still a small model.

- Very long transcripts may get shortened or lose details, so keep single runs reasonably short.
- Domain-specific terminology can be “over-corrected”. For technical audio, keep `refine=false` and edit yourself.
- Sometimes it removes too much. It can’t always tell whether a hesitation is meaningful.

## Wrap Up

Whisper “hears”, AFM “edits”. Together, you get clean transcripts in under two seconds, with no cloud dependency and no API cost. If you use voice input a lot, this pipeline is worth setting up.