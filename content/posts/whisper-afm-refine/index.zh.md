---
date: '2026-01-18T17:15:00+09:00'
title: '给 Whisper 加个本地「编辑」：用 Apple Foundation Model 自动清理语音转写'
slug: 'whisper-afm-refine'
categories: ['AI']
tags: ['Apple Intelligence', 'macOS', 'Whisper', 'Local AI', 'Productivity']
isCJKLanguage: true
cover:
  image: 'cover.png'
  alt: 'cover'
  relative: true
---

用过语音转文字的人都知道，转出来的文字往往不能直接用。口语里的「嗯」「那个」「就是说」会原封不动地出现在结果里，标点几乎没有，断句全靠猜。你要么自己改一遍，要么就忍着用。

最近折腾 Apple Foundation Model 的时候，我发现它特别适合干这个活：把语音转写的「口语稿」整理成「书面稿」。而且因为是本地模型，处理速度快、不花钱、数据不出设备。

## 问题：Whisper 转出来的东西太糙

拿一段日常语音举例，Whisper 转出来大概是这样：

> 今天打算做的事情还有哪些呢一会儿我觉得可以让AI助理帮我每天看一下我的Google Analytics里面我的Blog Site的访问情况毕竟我平时想不起来去看让他帮我追一下还有一件能做的事情就是就是什么来着怎么突然想不起来了那个算了先这样吧

没有标点，没有分段，重复的「就是就是」也保留着，口语词「那个」「什么来着」一个不少。这种文字发出去或者存档都不太体面。

## 方案：Whisper 转写完直接丢给 AFM 清理

Apple Foundation Model 虽然只有 3B 参数，但做文本润色这种事情绰绰有余。我让它扮演一个「编辑」角色，把口语整理成书面语：

```
System Prompt: 将语音转写的口语文字整理为书面语。去除口语词（嗯、啊、就是、那个等）、删除重复内容、修复断句、添加标点。保持原意，只输出整理后的文字。
```

同样那段语音，经过 AFM 处理后变成：

> 今天打算做的事情还有哪些呢？一会儿我想让AI助理每天查看我的Google Analytics，看看博客的访问情况。毕竟平时我常常忘记去看，让它帮我跟踪一下。还有一件事情，我突然想不起来了，算了先这样吧。

标点有了，断句清晰了，重复和口语词都删掉了，但意思完全没变。

## 实现：一个 API 搞定

我把 Whisper 和 AFM 打包成了一个 HTTP 服务。调用方式和原来的 Whisper API 一模一样，但输出已经是清理好的文字。

```python
# server.py 核心逻辑
@app.post("/asr")
async def asr(audio_file: UploadFile, refine: bool = True):
    # Step 1: Whisper 转写
    raw_text = transcribe_audio(audio_file)

    # Step 2: AFM 清理（可选）
    if refine:
        return await refine_with_afm(raw_text)
    return raw_text
```

关键点很简单：用 mlx-whisper 做本地转写，转写结果发给 AFM 的 `/v1/chat/completions`，加个 `refine` 参数，想要原始结果就关掉。

## 性能：额外开销不到 1 秒

实测一段语音：

| 模式 | 耗时 |
|------|------|
| 纯 Whisper | 1.05s |
| Whisper + AFM | 1.85s |

AFM 的处理大概增加 0.8 秒。对语音消息来说，这点延迟完全可以接受，换来的是可以直接用的文字。

## 部署：一个 launchd 服务

我把它做成了 macOS 的 launchd 服务，开机自启，监听 9000 端口。这样所有语音转写请求都会自动经过这层清理，调用方完全不用改代码。

## 适用场景

语音笔记、聊天记录、会议纪要、播客/视频字幕的预处理，这几类场景都很合适。

## 局限

AFM 毕竟是个小模型。

太长的文本可能会截断或漏掉内容，建议单次处理控制在 500 字以内。

专业术语有时会被「润色」错，如果你的语音涉及很多专业词汇，建议关掉 refine 自己改。

偶尔会过度删减，它分不清哪些「嗯」是真的废话，哪些只是有意义的停顿。

## 小结

Whisper 负责「听」，AFM 负责「编辑」，两个本地模型配合，就能把语音变成可以直接用的文字。整个过程不到 2 秒，不花钱，数据不出设备。

如果你经常用语音输入，这个流程值得搭一下。