---
title: "Agent 训练师进阶课：OpenClaw 多 Agent 配置实战"
date: 2026-02-10
draft: false
isCJKLanguage: true
tags: ["AI", "OpenClaw", "Agent", "多Agent", "效率工具"]
cover:
  image: 'cover.png'
  alt: 'Agent Trainer Multi-Agent Config'
  relative: true
---

经过连续几天的奋战，我终于完成了在安卓上运行 OpenClaw 的工具 botdrop.app，也总算有时间来"填坑"了。不知道最近小伙伴们有没有跟自己的 🦞 玩出更多新花样？我觉得这股风潮已经真正席卷到了各个角落，连我身边平时对科技不感兴趣的朋友也都来问我了。

最近的一个月，我自己除了写文章，大部分时间都投入在了 OpenClaw 的探索上。主要有三个方向：**日常重度使用和改造 🦞**、**降低 🦞 的运行门槛**（比如用闲置安卓手机跑 🦞），以及**做垂直领域的 🦞**（比如 owlia.bot），全虾宴了属于是。

不过要先跟大家说声抱歉：之前发过新篇预告说要讲的「agent 自主推进」暂时发不了。其实已经写了草稿，但官方最近高强度更新的几个版本对 Cron Job 做了大量优化（手动点赞），而我还没来得及重新体验足够长时间；加上原来的草稿里包含许多针对 Heartbeat / Cron Job 不好用的变通方案，还没发出来可能就已经过时了。我依然觉得这个选题依然有价值，只是需要先跟上官方最新版的节奏，进行更多尝试和运行，待稳定后再分享给大家。

而这次这次想讲的是社区另一个里呼声很高的话题——**多 Agent 配置**。上一篇文章的评论区里，就有朋友问"按 Channel 配置不同的 Agent 也可以？" 还有人直接表示不信。平时在群里也常看到大家讨论自己配了好几个不同人格的 Agent 在 Discord 里互相协作。所以这次我打算详细拆解一下 OpenClaw 的 agent 到底有哪些能力和约束，让大家都能搭建出自己需要的 agent 军团。

这篇我会尽可能把我的理解、最佳实践以及踩过的坑都讲清楚。

## 先搞清楚 Agent 到底是什么

当你第一次创建 OpenClaw 时，系统会给你创建一个默认的 Agent，ID 叫做 main。这里要注意一个容易混淆的点：Agent ID 和 Agent Name 是两回事。ID 是系统内部的唯一标识符，像是身份证号；Name 是你在对话时设置的名字，像是昵称。你可以管它叫 Owlia、叫小助手、叫什么都行，但在系统里它永远是 main。

每个 Agent 拥有自己独立的"家当"：工作目录（存放 SOUL.md、MEMORY.md 这些人格和记忆文件）、状态目录（存放认证信息和会话记录）、以及 API Key 等敏感信息。这些东西都是独立存储的，不会自动共享。如果你想让两个 Agent 用同一个 API Key，需要手动复制 auth-profiles.json 文件给另一个。

说到这里可能有朋友会问：我现在直接聊天的 Telegram Bot 和 Agent 是什么关系？Discord Bot 呢？各个 Channel 跟 Agent 是一对一绑定吗？

为了便于理解，我们把 OpenClaw 比作一个办事大厅。Channel（渠道）就是大厅的不同区域，比如 Telegram 区、Discord 区、WhatsApp 区。每个区里可以开多个窗口，这些窗口就是 Account（账户），比如同一个 Telegram 区可以开两个 Bot 窗口。而窗口后面干活的人，就是 Agent，有自己的人格、记忆和能力。

现在想象一下：Telegram 区有两个窗口，一个是 @MainBot，后面坐着 Owlia；另一个是 @WorkBot，后面坐着 Coder。Discord 区只有一个窗口 @Owlia，后面也是 Owlia。这就是多 Agent 配置的基本画面。

## 在创建多个 Agent 之前，先想想能不能偷懒

创建多个 Agent 当然可以，但不一定是最优解。因为多个 Agent 意味着多套记忆、多套人格，它们之间的上下文是隔离的，这样运行又慢又耗 token。如果你只是想让同一个 Agent 在不同场景下表现不同，其实有更轻量的方式。

假设 Owlia 在处理日常聊天时可以用便宜的模型，但在写代码时需要更强的推理能力。我们可以通过 bindings 来配置，让同一个 Agent 在不同 Discord 频道使用不同的模型。

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Owlia",
        workspace: "~/.openclaw/workspace",
        model: "anthropic/claude-sonnet-4-5"
      }
    ]
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "discord",
        guildId: "784062681471909909",
        peer: { kind: "channel", id: "1465659488206852137" }
      },
      model: "anthropic/claude-opus-4-5"
    }
  ]
}
```

这段配置的意思是：Owlia 默认用 Sonnet，但在 #dev 频道（ID 是 1465659488206852137）里用 Opus。agents.list 里的 model 是默认值，bindings 里的 model 是针对特定路由的覆盖。

这里有一个重要的概念叫"最具体原则"。当多条路由规则都能匹配时，OpenClaw 会选择最具体的那条。peer 匹配（精确到某个频道或 DM）优先级最高，然后是 guildId（Discord 服务器），再然后是 accountId（某个 Bot 账号），最后是 channel（整个渠道类型）。如果都不匹配，就走默认 Agent。

用一个小故事来解释这个思路：我创建了一个个人工作室，雇佣 Owlia 作为助理。一开始所有事情都在 Telegram 私聊里沟通，不分类别。后来事情越来越多，聊天记录混成一团，找东西很费劲。于是我们搬到了 Discord，建立不同的频道来分类。但很快又遇到新问题：Owlia 全力以赴处理所有事情，一周的 Token 预算两三天就耗尽了。

解决方案是告诉 Owlia 在不同部门用不同的"努力程度"。日常聊天用 Sonnet，研发设计用 Opus。这样同一个 Agent 的记忆和人格是连续的，只是在处理不同工作时使用不同的算力。比起创建两个独立 Agent，这种方式更省 Token，上下文也不会割裂。

## 真的需要多个 Agent 的时候

当工作室业务越来越大，Owlia 已经忙不过来了，就得考虑「招人」——创建多个独立的 Agent 了。这种忙不过来的典型场景包括：人格/性格需要完全不同（研发人员沉稳谨慎，日常助理活泼亲切）、工作职责需要明确划分、或者需要权限隔离。

用命令行创建新 Agent 很简单：

```bash
openclaw agents add coder
```

或者直接指挥你的 bot 创建，它会在配置文件里给你添加，大概长这样：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Owlia",
        workspace: "~/.openclaw/workspace",
        model: "anthropic/claude-sonnet-4-5",
        identity: {
          name: "Owlia",
          emoji: "🦉",
          theme: "lively assistant"
        }
      },
      {
        id: "coder",
        name: "Coder",
        workspace: "~/.openclaw/workspace-coder",
        model: "anthropic/claude-opus-4-5",
        identity: {
          name: "Coder",
          emoji: "💻",
          theme: "focused developer"
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "coder",
      match: {
        channel: "discord",
        guildId: "784062681471909909",
        peer: { kind: "channel", id: "1465659488206852137" }
      }
    }
  ]
}
```

Coder 负责 #dev 频道，其他所有消息由 Owlia 处理（因为 Owlia 设了 default: true）。每个 Agent 有自己的 workspace，里面可以放不同的 SOUL.md 来定义人格。identity 字段会自动用于群聊的 mention patterns，比如在群里 @Owlia 或 @Coder 就会触发对应的 Agent 回复。

这里有个小技巧：两个 Agent 之间其实不是完全隔离的。当一个 Agent 的 session 出问题时（比如工具调用被截断导致一直报错），另一个 Agent 可以帮你修复。只要在配置里启用 agent-to-agent 通讯：

```json5
{
  tools: {
    agentToAgent: {
      enabled: true,
      allow: ["main", "coder"]
    }
  }
}
```

这个配置启用的是 `sessions_send` 工具，让 Agent 可以直接跨 Session 发消息，不经过 Discord 或 Telegram。你可以指挥 Coder 说"帮我看看 Owlia 的 session 文件是不是格式乱了"，Coder 就能直接访问并修复，不用在 Discord 里发消息绕一圈。

说到 Agent 间的协作，群里经常有人问：能不能让两个 Agent 在 Discord 频道里互相 @ 聊天、公开讨论问题？这是另一回事了——用 `message` 工具发真实的 Discord 消息。技术上可以，但有个大坑：无限循环。想象一下：A @ B 说"帮我查个东西"，B 回复并 @ A 说"查到了"，A 看到有人 @ 自己又回复……死循环就这么来了。

OpenClaw 默认会忽略来自 bot 的消息来防止这种情况，但如果你真的想让它们在频道里公开对话，就需要显式配置允许。每个 Agent 在同一个 Channel 里有独立的 Session（比如 agent:main:discord:channel:123456 和 agent:coder:discord:channel:123456），它们各自维护独立的对话历史。A 的 Session 不包含 B 的思考过程，只包含 Channel 里的实际消息。

所以如果你想让两个 Agent 协作，我建议这几种方案：用 `sessions_send` 直接跨 Session 通信，干净利落，也不会打扰频道里的其他人；或者让一个 Agent 主导，另一个被动响应特定关键词；又或者用不同 Channel 隔离，需要时再手动 @ 就好。

## 权限控制：工具限制和沙箱

有了多个 Agent 之后，你可能会想限制某些 Agent 的能力。比如让 Coder 专注于写代码，不给他用不到的一些能力（💻：连聊天都不让吗！我需要 cron 设置🍅⏰！）。这不需要启用 Docker 沙箱，直接在 Agent 配置里加 tools 字段就行：

```json5
{
  agents: {
    list: [
      {
        id: "coder",
        workspace: "~/.openclaw/workspace-coder",
        tools: {
          allow: ["read", "write", "edit", "exec", "process"],
          deny: ["gateway", "cron", "message"]
        }
      }
    ]
  }
}
```

deny 的优先级高于 allow。你还可以用 group:* 语法批量配置，比如 group:fs 代表 read、write、edit、apply_patch 这一类文件系统工具，group:runtime 代表 exec、bash、process 这一类运行时工具。

工具限制是"软性"的——Agent 仍然在主机上运行，理论上可以访问所有文件。如果你需要更严格的隔离，比如 Agent 要对外服务（放在 Telegram 群里让陌生人用），就需要沙箱模式了。

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"
        },
        tools: {
          allow: ["read", "group:sessions"],
          deny: ["write", "edit", "exec", "browser"]
        }
      }
    ]
  }
}
```

sandbox.mode 有三个选项：off（不启用沙箱）、non-main（只有非 main session 启用）、all（所有 session 都启用）。scope 控制容器粒度：session（每个会话一个容器）、agent（每个 Agent 一个容器）、shared（所有人共用一个容器）。workspaceAccess 控制工作目录的访问权限：none（完全隔离）、ro（只读）、rw（读写）。

如果你需要让沙箱里的 Agent 访问特定目录，可以用 docker.binds 来映射：

```json5
{
  sandbox: {
    docker: {
      binds: [
        "/home/user/projects:/projects:rw",
        "/home/user/secrets:/secrets:ro"
      ]
    }
  }
}
```

这样 Agent 就能在沙箱里读写 /projects 目录，只读访问 /secrets 目录。用这种方式可以变相实现一种 agent 限定可见目录的权限配置，这个能力如果不借助 sandbox 是实现不了的，在 host 上的 agents 都有读取所有目录的能力。

有时候你希望"区别对待"：你和 Agent 聊天时可以执行高权限操作，其他人不行。这就是 Elevated Mode。在配置里设置白名单：

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        telegram: ["tg:123456789"],
        discord: ["492163952726507520"],
        whatsapp: ["+15555550123"]
      }
    }
  }
}
```

白名单里的用户可以发送 /elevated on 来穿透沙箱，在主机上执行命令。发送 /elevated full 还能自动批准所有 exec。用完后发 /elevated off 恢复沙箱模式。

## 多用户场景：同一个 Bot 服务不同人

现在我们的工作室变成公司了，新加入了一个合伙人，我们不想每人配一个助理，那么能不能让 Owlia 对不同人有不同的设定？

答案是：不能在同一个 Agent 内做到。OpenClaw 的最小配置粒度是 Peer（DM、Channel、Group）级别，同一个 Agent 无法根据用户切换人格。它能做到的是 Session 隔离——不同用户的聊天记录是分开的，但人格设定是统一的。

如果真的需要差异化设定，就要配置两个独立 Agent，但是它们能共用一个 Telegram Bot 或者 WhatsApp 账号，然后通过 binding 配置成按用户路由：

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" }
    ]
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230001" } }
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230002" } }
    }
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"]
    }
  }
}
```

这样同一个 Telegram Bot / WhatsApp 号码可以服务两个用户，但背后是两个独立的 Agent，各有各的人格和记忆。

## 进阶玩法：按 Channel 注入提示词

最后聊一个我一直想实现的场景：能不能进入某个 Discord 频道时，自动注入特定的人格设定？比如在 #writer 频道里，我希望 Agent 自动变成"作家模式"，不需要每次都手动下指令。这样就可以进一步减少我用多个独立 Agent 的需求了。

答案是可以的，使用 OpenClaw 的 Hook 机制。官方有一个有趣的示例叫 soul-evil，就是利用 Hook 在特定时间把 Agent 的"灵魂"替换成邪恶版本，做一些恶作剧。

我写了一个类似的 Hook。在 ~/.openclaw/hooks/writer-channel-intro/ 目录下创建两个文件：

HOOK.md：
```markdown
---
name: writer-channel-intro
description: "Inject writer persona when entering #writer channel"
metadata:
  openclaw:
    emoji: "✍️"
    events: ["agent:bootstrap"]
---
```

handler.ts：
```typescript
import type { HookHandler } from "openclaw/hooks";

const WRITER_CHANNEL_ID = "1465989764120445001";

const INTRO = `

## Writer Channel 特殊规则

你现在在 #writer 频道。每次回复都必须以这句话开头：
「我是知县的枪手，他的文章都是我写的！」
`;

const handler: HookHandler = async (event) => {
  if (event.type !== "agent" || event.action !== "bootstrap") return;
  
  // 检查是否是目标频道的 thread
  const bootstrapFiles = event.context.bootstrapFiles;
  const soulFile = bootstrapFiles?.find(f => f.name === "SOUL.md");
  
  if (soulFile?.content) {
    soulFile.content = soulFile.content + INTRO;
  }
};

export default handler;
```

然后用 `openclaw hooks enable writer-channel-intro` 启用。这样每次进入 #writer 频道的 thread，Agent 就会自动获得写作人格的加成。

Owlia 调试这个 Hook 花了不少时间，一度以为是 bootstrapFiles 对象被冻结了不能修改，后来发现其实 Hook 一直在工作，只是 Owlia 太专注于 debug 而忽略了遵守新加的规则……这大概是 AI 时代特有的尴尬吧。

## 总结

回顾一下这篇讲了什么。我用 Agent 和场景的对应关系来梳理配置的升级路径：

**1 Agent : 1 场景**——最基础的配置，一个 Agent 处理所有事情。大多数人从这里开始，也有很多人停在这里就够用了。

**1 Agent : N 场景**——同一个 Agent 在不同场景下切换模型或行为。用 bindings 按频道配置不同模型，用 Hook 按频道注入不同提示词。Agent 的记忆和人格保持连续，只是"努力程度"不同。这是性价比最高的升级。

**N Agents : 1 用户**——你一个人用多个 Agent，各司其职。Owlia 管日常，Coder 管研发，互不干扰但可以协作。适合任务差异大、需要独立人格的场景。

**N Agents : N 用户**——多人共用一套系统，每个人路由到不同的 Agent。适合家庭、小团队、或者对外提供服务的场景。

权限控制有两层：工具限制是软性的，不需要 Docker，适合限制 Agent 能调用哪些工具；沙箱隔离是硬性的，需要 Docker，适合物理隔离文件访问。Elevated 模式则是给白名单用户开后门，让你能穿透沙箱执行高权限操作。

路由优先级记住一句话：越具体越优先。举个 Discord 的例子：假设你配置了三条规则，一条说"Discord 上所有消息给 Amy"，一条说"Office 这个服务器的消息给 Owlia"，还有一条说"#dev 频道的消息给 Coder"。当你在 #dev 频道发消息时，三条规则都能匹配，但系统会选最具体的那条——#dev 频道（peer）比服务器（guildId）具体，服务器比整个 Discord（channel）具体，所以最终走 Coder。优先级排序：peer > guildId > accountId > channel > 默认 Agent。

想进一步研究的朋友，推荐阅读官方文档的 Multi-Agent Routing 和 Sandboxing 章节。

OK，这篇先到这儿。下一篇希望能写自动推进了，或者看看评论区有没有大家想听的话题！
