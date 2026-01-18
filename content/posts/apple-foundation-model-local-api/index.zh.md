---
date: '2026-01-18T13:20:00+09:00'
title: '白嫖 Apple Intelligence：把 macOS 内置模型变成 OpenAI API'
slug: 'apple-foundation-model-local-api'
categories: ['AI']
tags: ['Apple Intelligence', 'macOS', 'LLM', 'Local AI', 'OpenAI API']
isCJKLanguage: true
---

macOS 26 Tahoe 内置了一个 3B 参数的语言模型，Apple 叫它 Foundation Model。这玩意儿平时藏在系统里给 Writing Tools、Siri 这些功能用，但其实你可以把它拉出来，包装成一个 OpenAI 兼容的 API，然后接到任何支持 OpenAI 格式的工具里去。

完全本地运行，不走网络，不要钱。

## 前提条件

你需要 macOS 26 Tahoe（现在已经是 26.3 正式版了）和一台 Apple Silicon 的 Mac。Intel Mac 不行，因为 Apple Intelligence 只支持 M 系列芯片。

## 安装

有个开源项目叫 maclocal-api，做的事情很简单：调用 Apple 的 FoundationModels framework，然后在本地起一个 HTTP 服务器，暴露 `/v1/chat/completions` 这些标准端点。

```bash
git clone https://github.com/scouzi1966/maclocal-api
cd maclocal-api
swift build -c release
```

编译需要一点时间，因为依赖了 Vapor 框架，会拉一堆 Swift 包下来。我这边大概两分钟编译完。

编译完之后，二进制文件在 `.build/release/afm`。

## 启动服务

```bash
./.build/release/afm --port 9999 --hostname 0.0.0.0
```

会看到一个很花哨的 ASCII art logo，然后服务就起来了。默认监听 9999 端口。

如果想后台运行：

```bash
nohup ./.build/release/afm --port 9999 --hostname 0.0.0.0 > /tmp/afm.log 2>&1 &
```

## 使用

API 完全兼容 OpenAI 格式，所以你可以直接用 OpenAI 的 SDK：

```python
from openai import OpenAI

client = OpenAI(
    api_key="not-needed",
    base_url="http://localhost:9999/v1"
)

response = client.chat.completions.create(
    model="foundation",
    messages=[{"role": "user", "content": "你好"}]
)
print(response.choices[0].message.content)
```

或者 curl：

```bash
curl http://localhost:9999/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "foundation", "messages": [{"role": "user", "content": "Hello"}]}'
```

支持 streaming，加 `"stream": true` 就行。

## 实测能力

我跑了一圈测试，结论是：能用，但别期望太高。

**能做的事：**

中英翻译没问题。我让它把「今天天气真好」翻成英文，输出 "Today, the weather is really good."，挺自然的。

写代码也行。让它写个斐波那契函数，Python 代码结构清晰，还带注释和使用示例。

简单推理可以。「小明比小红大3岁，小红今年10岁，小明几岁？」答「13岁」，没毛病。

System prompt 能遵循。我让它扮演一只猫，它真的回复「喵喵喵喵」。

摘要、改写这些 Writing Tools 本来就在用的功能，自然也没问题。

**做不了的事：**

没有 function calling / tool use。这是个纯文本模型，不能调工具。

复杂推理会翻车。毕竟只有 3B 参数，比不上 Claude 或 GPT-4。

不能生成图片。Apple 的图片生成是另一套东西（Image Playground），这个 API 没暴露。

Vision 是假的。maclocal-api 有个 `afm vision` 命令，但那只是调用 Apple 的 OCR framework 提取文字，不是真正的多模态理解。

## 资源占用

这是最惊喜的部分。

afm 服务器本身只占 33MB 内存，CPU 空闲时 0%，请求时 0.7% 左右。

为什么这么轻？因为模型推理不是这个进程干的。Apple 的 Foundation Model 由系统服务 `IntelligencePlatformComputeService` 托管，这个服务本来就一直在后台跑（给 Writing Tools 用），所以你启动 afm 几乎没有额外开销。

换句话说，模型早就加载好了，afm 只是个薄薄的 API 包装层。

## 适用场景

这东西适合什么？

本地隐私场景。数据完全不出机器，如果你在处理敏感内容，这个方案比任何云端 API 都安全。

简单任务的免费替代。翻译、摘要、简单问答，用这个就够了，省点 API 费用。

离线环境。飞机上、没网的时候，照样能用。

不适合什么？

复杂的 AI agent 流程。没有 tool calling，没法做复杂编排。

需要高质量输出的场景。3B 模型的天花板在那里，别指望它写出惊艳的文案。

## 和 Ollama 的区别

你可能会问，这和 Ollama 有什么区别？

Ollama 需要你自己下载模型，占几个 G 到几十 G 的硬盘空间。Apple Foundation Model 是系统自带的，不占额外空间。

Ollama 的模型你可以换，想跑 Llama 3 就跑 Llama 3，想跑 Qwen 就跑 Qwen。Apple 这个只有一个 3B 的 foundation 模型，没得选。

性能上，Apple 的模型针对自家芯片做过优化，推理速度很快。但 3B 参数的能力上限摆在那里，如果你用 Ollama 跑个 70B 的模型，效果肯定比这个好。

所以这两个不是替代关系，更像是互补。轻量任务用 Apple 的，重活用 Ollama 跑大模型。

## 小结

Apple 在 macOS 26 里埋了一个免费的本地 LLM，虽然能力有限，但胜在零成本、零配置、零隐私顾虑。如果你是 Mac 用户，值得花五分钟装一下，至少可以拿来做 OCR 和简单的文本处理。

GitHub 地址：https://github.com/scouzi1966/maclocal-api
