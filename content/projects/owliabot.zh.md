---
title: "OwliaBot"
description: "安全加密原生 AI Agent"
date: 2026-01-01
externalUrl: "https://owlia.bot"
showDate: false
showReadingTime: false
build:
  render: false
  list: true
---
安全加密原生 AI Agent，内置钱包方案，密钥隔离设计。

- 密钥与 Agent 进程隔离 — 即使被 RCE 也无法窃取密钥
- 签名分级风控 — 根据金额和频率设置审批阈值
- 执行环境收窄 — 命令白名单，无 shell 执行
