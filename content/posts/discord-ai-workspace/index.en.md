---
date: '2026-01-16T09:19:23Z'
title: 'My "Doomsday Cabin": Building an AI Workbench with Discord'
slug: 'discord-ai-workspace'
categories: ['AI']
tags: ['discord', 'ai', 'productivity', 'automation']
cover:
  image: cover.jpg
---

If you're someone actively exploring the AI era, you should have your own Discord Server.

I call my Discord Server the "Doomsday Cabin." The name might sound a bit dramatic, but it's genuinely my most relied-upon work environment right now. "Doomsday" is a mental anchor I set for myself—the imagery helps me imagine being in the quiet of a wilderness, having one place I know still works, storing everything important to me. No matter how noisy or fast-changing things get outside, I still have a quiet place to continue working, thinking, and iterating.

![A doomsday tracked vehicle by Bilibili creator "漠生铁铁"—what guy wouldn't want one!](doomsday-vehicle.jpg)

The AI era has a very practical problem: there are too many tools. Doing anything slightly complex might mean jumping between a dozen apps, each with its own account, payment model, and data format. In this state, AI can't really help because it can't see the whole picture. When context is fragmented, AI can only do fragmented things.

So I started wondering: is there somewhere I could put everything together? Not some bloated "all-in-one" product, but a flexible enough base where I can plug things in, pull things out, and swap things at will. Like a "breadboard" that electronics hobbyists use—put components on it and they run; if it doesn't work, swap it out.

![A breadboard with various components](breadboard.jpg)

Discord happens to be exactly that.

## Why Discord

It's free—creating a Server costs nothing. It has a complete Bot API that can do far more than most people imagine. Its internal structure is rich enough: Category, Channel, Message, Thread—naturally a hierarchical knowledge container.

Webhooks are magical. Anything that can send HTTP requests can post messages to a channel without running a Bot. GitHub commits, Notion updates, server alerts—a few lines of code and they're integrated.

Discord also has Forum-type channels where each post comes with its own discussion area, perfect for issue tracking or idea collection. More structured than regular Threads, with tagging and sorting capabilities.

The permission system is granular. You can give different Bots different permissions, set different access rules for different channels. If you want to open some features to friends later, you don't have to worry about them seeing things they shouldn't. Search works too, supporting from, in, before, after filters—finding historical messages is convenient.

As a mature product, it has everything you'd expect: multi-device sync, account system, security, permission management. I don't have to worry about any of this infrastructure. Want to share with others? Invite them in. Don't want to share? Use it alone. A one-person Server works perfectly fine.

Could Telegram do something similar? Yes, but its structure isn't as rich, and what Bots can do is limited. Discord is more like an open development board—the platform exposes many internal capabilities, messages can have buttons and forms, and Bots can have complex interaction flows.

## How I'm Using It

Here are a few examples of how I use it.

Every morning, the system automatically scans all my subscription feeds, runs them through an LLM for summaries, then posts a message in a dedicated channel. A Thread opens below the message with each item's summary posted sequentially. When I wake up and glance at it, I know what's worth reading today.

![Daily Digest](daily-digest.jpg)

There's also voice-to-text. I record a voice message and send it to a channel, and the Bot automatically opens a Thread below the message with the transcription. These results have already been lightly polished by an LLM, so I don't need to worry about colloquial expressions or pauses when recording. Whenever I have inspiration to write something, I just speak to Discord on my phone, and after automatic queued transcription, the material is there within minutes.

![Voice Transcription channel](voice-transcription.jpg)

Reactions can be used creatively too. If I add a red star to a message, it automatically forwards to a favorites channel. An eye icon means "dig deeper"—the Bot sends it to an LLM for Deep Research and posts the results back. English content also gets translated to Chinese. The whole flow triggers with one click, no switching apps, no copy-pasting.

![Reaction Quick Actions](reaction-workflow.jpg)

## What Else Is Possible

This is just the beginning. The Discord base can grow into many things.

**Personal CRM**. Create a channel dedicated to recording interactions with people. Each person gets a Thread—what you discussed, what you promised, what to follow up on next. The Bot can periodically remind you who to contact.

**Automated Journaling**. Every night the Bot asks you a few questions: what did you do today, any thoughts, what's planned for tomorrow. After you reply, it automatically organizes it into a diary archive. Over time, let AI analyze your state trends. [I've already built a version of this](https://x.com/zhixianio/status/2011822312275026426)—it's pretty good.

**Reading Management**. Drop a link in, and the Bot automatically fetches content, generates summaries, and adds tags. Use Reactions to mark "read," "dig deeper," "archive." Much more flexible than tools like Pocket because you define the rules.

**Code Snippet Library**. Send frequently used snippets to a dedicated channel, and the Bot automatically identifies the language, adds syntax highlighting, and tags. Search when you need something and it's there.

**Voice Channels can go even further**. Not just voice chat—you can integrate AI real-time voice conversation. Enter a voice room, speak directly to AI, and it responds in real-time, like a phone call.

**Multi-Agent Collaboration**. Different Bots handle different things, and they can @ each other to trigger actions. For example, an information-gathering Bot finds important news, @s an analysis Bot for deep interpretation, then @s a notification Bot to push to you. A small Agent cluster running in your private channels.

At its core, Discord is a canvas with complete permissions, rich UI components, and mature infrastructure. Draw whatever you want on it; if you're not satisfied, erase and start over. Go build your own Server, install a few Bots, connect them to LLMs, and as you use it, you'll find this little cabin can grow into many things you didn't anticipate.
