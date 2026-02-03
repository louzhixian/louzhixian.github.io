---
title: "Agent 训练师进阶指南：用 Discord 打造高效 OpenClaw 协作系统"
date: 2026-02-03
draft: false
isCJKLanguage: true
tags: ["AI", "OpenClaw", "Agent", "Discord", "效率工具"]
cover:
  image: 'cover.png'
  alt: 'Agent Trainer Discord Guide'
  relative: true
---

说好的第二篇教程来了。没想到短短几天没更新，Moltbot 又改名了，只能说 AI 时代的节奏真是太快了。比改名更快的是 moltbook 掀起的风潮，各种面向 Agent 的产品层出不穷，让人眼花缭乱，我也是脑洞大开，看到了新时代的雏形。不过这一期咱们还是先不谈这些，先把之前的坑填上，分享一下我日常使用通过 Telegram 和 Discord 使用 OpenClaw 的经验和心得。

## 渠道选择：WhatsApp、Telegram 还是 Discord？

OpenClaw 目前支持的主流渠道包括 WhatsApp、Telegram、Discord、iMessage、Slack 等，我实际使用过的是 WhatsApp、Telegram 和 Discord。

首先，如果没有特别必要，建议先排除 WhatsApp，它的使用成本和隐患都比较大。这是因为 OpenClaw 登录的方式并非官方允许的途径，本质上是把 OpenClaw 当作网页客户端扫码登录，有点 hack 的意思，而且这会导致连接不稳定，经常断联。此外，这种方式大概率需要准备一个单独的手机号来注册，毕竟不能用你现在的 WhatsApp 账号登录，要不然你用什么跟它交互😂。所以，如果不是特别依赖，我建议可以放弃这个渠道。

然后就是 Telegram 和 Discord 两兄弟了。它们的调性区别很明显：Telegram 偏向点对点的 DM 聊天方式；Discord 以服务器（Server）为核心，里面包含大量频道和自由度，这种特性能够给 OpenClaw 更多的发挥空间。在我的搭配中，Telegram 主要负责"短平快"的直接聊天，Discord 则主要用于处理复杂、成体系的任务，以及可以并行推进的任务。

## 理解 Main Session：你的 AI 助手如何识别主人

不过，在开始讲我自己的 Setup 之前，我需要先讲一下 OpenClaw 的 Main Session 概念。

Main Session 就是 OpenClaw 认为它跟主人（Owner）直接对话的那个 Session。比如 Telegram、WhatsApp、Discord Bot 的 DM，还有 iMessage，这种你跟 Bot 一对一直接聊的场景，在它看来就是直接跟 Owner 讲话。它的一个好处是，主记忆文件 MEMORY.md 可以默认加载进来，因为这个文件里存的是有隐私属性的上下文信息。其他 session 里就不会加载，比如在群里聊天，其他人随便问问就把主人喜欢看的动漫类型给套出来，这就不太好了哈哈。

另外一个好处是，它默认这些 Session 是打通的。也就是你在 iMessage、Discord、Telegram 里分别说 1、2、3，在 Agent 看来其实就是在一个聊天里说了 1、2、3，没有区别。

但这也带来一个可能的问题：如果你把 Bot 分享给家人朋友使用（比如开放了 Telegram 访问权限），Agent 会无法区分是你发的消息还是家人发的消息，都算在同一个上下文里，这样就会乱掉。

有一个配置可以解决这个问题，就是 session.dmScope。把它的值配置成 "per-channel-peer"，就可以实现不同 Channel、以及同一个 Channel 不同用户的隔离。这样无论是你自己的 Discord、Telegram 之间，还是你跟家人的 Telegram 之间，在 agent 那里都会变成独立的 Session，互不干扰。

```json
...,
"session": {
    "dmScope": "per-channel-peer"
},
...
```

那你可能会想，能不能既跟家人分开，又不影响我自己 DM 之间的互通呢？这也是可以的。因为还有一个配置叫 session.identityLinks，在这里把你自己的不同 DM Session 链接成同一个，就能实现"对外分割，对内互通"的状态。

```json
session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
   ...
}
```

刚刚说的这些配置，大家不需要手动去改配置文件，因为容易改坏。最好是直接给你的 OpenClaw Bot 下令，让它去配置。

更多相关配置可以参考官方文档的[这个部分](https://docs.openclaw.ai/gateway/security#dm-session-isolation-multi-user-mode)。

## Discord 的信息分级：Thread 的魅力💬

Discord 还有一个特别好的信息分级能力，我觉得是借鉴于 Slack，那就是 Thread。你可以针对任何一条消息 Create Thread，在这个消息下面继续讨论相关主题。比如一个 Channel 叫「中午吃什么」，大家每天都会在里面讨论想吃的菜。如果今天小帅说「我想吃鱼」，下面大家你一言我一语地回复，有的继续讨论鱼相关的话题比如「烤鱼还是炖鱼？」，有的人新起一个话题说「我想吃牛肉」。这其实就是微信群的现状（当然有了 Quote 功能后好了一些），虽然可以交流，但容易让整沟通变得混乱。

如果是 Thread 模式呢？如果你想就吃鱼的话题继续讨论，就在小帅的消息下创建一个 Thread，这时候会新开一个聊天空间，大家就可以顺着「吃什么鱼」的话题聊下去。而在 Thread 之外，依然是大家提议今天中午吃什么的主线聊天，比如吃牛肉的人就在 thread 里已经开始订位子了。不难发现这是一个更好的交流架构，主题明确，细节可控。

介绍完 Thread 聊天模式，我们就可以把整个架构讲清楚：Discord Server 里每一个 Channel 都是独立的 Session，这很好理解；除了 Channel 之外，其实每一个 Thread 和 Channel 其实是一样的，都是一个独立的 Session。不过写文章时的最新版（2026.2.1）更新了一个功能：Discord: inherit thread parent bindings for routing. 简单说就是当 Thread 创建时，agent 会自动继承 parent channel 的最近消息作为上下文，让 agent 在 thread 里也能知道「前情提要」。用好 Thread 功能，你的 Discord 里信息组织性会大幅提高，session 爆炸的问题也不太会出现了。

## 我的 Discord 工作流：末日小屋实战

接下来展示一下我的 Discord 使用方式。我的用法是创建一个专门的 Server，看过我之前文章的小伙伴可能知道，我有一个 AI 工作流的 Server 叫「[末日小屋](https://x.com/zhixianio/status/2012092307613622525)」。现在我在这里专门开了一个 Section，用来承载我和 Owlia 🦉（我的 OpenClaw agent）工作的整个流程。

大家可以看一下截图，我基本上分了类：

一类叫"daily"，相当于头脑风暴阶段的聊天，可以看到下方有一大串还处于 open 状态的 Threads，在前面那张截图就是这个 channel 里的内容形式；另一种是已经独立出来、作为项目继续推进的内容，比如下面的 owliabot、writing；还有一类是日常推送，比如 digest 和 heartbeat。

在日常聊天频道里，我会给 Agent 一个指示：只要我在里面说一句话，你的回复一定要先创建一个 Thread（线程），然后在 Thread 里继续。目前这还是一个软约束，但运行得很好，基本都能遵循。

这样做的好处是，Daily 频道里的事情比较杂，当你突然想起一件事想继续讨论——比如"我最近想抄底 BTC 该怎么操作"——发完这条消息后，Agent 就会分析现状，询问你的偏好和目前仓位，然后你们就可以聊下去了。

当一件事变得相对庞大，你觉得可以变成长期项目，甚至可以让 Agent 主动推进时，就把它独立成一个 Channel。你可以在新频道里告诉 Agent 这是之前某个 Thread 的延续，让它把原来的 Session 压缩（Compress）到这边来，这样对话就能接上。接下来的推进会更轻松，因为在这个频道里，你依然可以用 Thread 的方式进行多维度的讨论和折叠。

这种工作方式的好处是我可以非常快速地在不同事务中切换。OpenClaw 默认能同时在 4 个不同的 Session 里回复消息，而且这个配额是可以改的，但对我来说 4 个足够了，我目前还很少能让 4 个频道同时处于回复状态（还得练）。这时你会发现自己就像"章鱼博士"，挥舞着四条大爪子操作四个线程的工作，感觉特别有成就感。

除了推进任务的 Channel，还可以建立一些日常定时任务的 Channel。这样之前在 main session 里的工作就可以拆分到不同的方向上。比如，我有一个专门的 Channel 叫 digest，它每天会定时跑几遍，把 X 和博客上的内容汇总发进来。再比如 heartbeat，我需要一个观察窗来了解它触发的时间和内容，所以建了一个 heartbeat Channel，让所有的输出都导向这里。大家可以结合日常工作生活的具体需求去配置。

配置好之后，整个"末日小屋"的能力会变得非常强。不仅拥有自动化的流程，屋子里还住进了一个非常强大的智能体，它可以帮你做事、查资料、聊天。这些 session 互相隔离，同时你还可以指定某个 session 去读取另一个 session 的内容，来快速同步上下文。这种体验非常契合推进复杂事务时的需求。

理论上 Slack 也可以做到，但我现在觉得以后可能连 Slack 都不用了，因为 Discord 的体验实在是太好了。

## Discord 进阶玩法：Reaction、多 Agent 与模型分配

Discord 还有一些进阶的玩法：

**Reaction**。你可以规定不同的 Reaction 代表不同的功能，让你的 Agent 按照你的操作去做相应的事情。比如收藏：你可以点一个红心♥️，它就自动把这个消息转发到某一个 channel 里面收藏。

**分 Agent**。每个 Channel 甚至 Thread 都可以配置成一个单独的 agent，可以有自己的各种设定，这样如果你需要多 agent 配合，不用非得创建多个独立 agent bot 拉一堆机器人进来，而是在 #product 里创建一个 pm bot，在 #dev 里创建一个 engineer bot 就好，甚至可以根据任务属性不同，给它们分别配置不同的模型，这样产品工作可能需要 opus 来做深度设计，开发工作很多时候 sonnet 就已经做得足够好了（当然复杂逻辑可能得配 codex 哈哈）。

## Telegram 的类似尝试（以及为什么我放弃了）

最后补充一些 Telegram 的类似用法，一个是开 Group 分 Topic，这个其实算是 Telegram 在 Discord 的压力下做的功能，但是一直体验很差；另外跟 Bot 的 DM 里也是可以开启 Thread（其实跟 Topic 一套机制），而且 1 月 24 日（当时还叫 Clawdbot）的那次更新加了一个支持：当你在 BotFather 那边给 Bot 设置打开 Thread 模式（注意必须在 BotFather 的 Mini App 里才能找到这个开关），你跟它的 DM 就会变成像一个带 Topic 的 Telegram Group，只不过 Topic 在 DM 里叫 Thread。我用下来体验很差，这里就不展开了。

## 用自然语言编程的时代

最后说一个感想。Reaction 自动化这个功能我之前在自己的"末日小屋"里是用写程序的方式实现的（当然也是 AI 写的），有了 Agent 之后，这些事情都可以用自然语言的方式「编译」到流程里面。这也是我觉得 Agent 时代跟原来最大的区别：人真的实现了用自然语言编程，只不过方式不是将自然语言翻译成程序代码，而是由一个既能听懂人话、又精通编程或者说调用接口的 Agent 来做中间的桥梁。

这个范式的变化，与当年的 AI 范式变化有异曲同工之妙。毕竟后来谁也没想到，现代大语言模型的核心架构 Transformer 竟然是从机器翻译这条路走出来的。

---

## 所以，Telegram 还是 Discord？

Telegram 轻快，适合随时随地问点事；Discord 重但组织能力强，适合长期项目。所以如果你只是想有个能随时聊两句的 AI 助手，Telegram 够用。但如果你想让 Agent 真的帮你干活、推项目、搞自动化，Discord 的 Server + Channel + Thread 三层结构会让你觉得"终于有个像样的工作台了"。我现在的用法是两边都开着，Telegram 当移动端入口，Discord 当主战场。

OK，这篇先到这儿，下一篇可能会写写定时任务和多 Agent 协作，或者看看评论区有没有大家想听的话题！
