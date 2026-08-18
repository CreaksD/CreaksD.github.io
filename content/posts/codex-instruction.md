---
title: "Codex 架构速览：Conversation、Agent、Tools 与 AGENTS.md"
date: 2026-08-18T10:30:00+08:00
draft: false
tags: ["Codex", "AI 编程", "AGENTS.md"]
---

OpenAI Codex（CLI 版）的整体架构可以概括为：一次对话由 **Conversation → Agent → Tools** 三层组成。

```
                         Codex
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
     Conversation        Agent             Tools
        │                  │                  │
        ↓                  ↓                  ↓
      Fork/Side          Plan             Terminal
      Chat/Archive       Mode             Git
                                           MCP
                                           Skill
```

## 三层架构分别是什么

**Conversation（对话层）**：负责管理一轮完整的任务对话，支持：

- **Fork**：从某条历史消息分叉出一条新的对话线，相当于「平行时间线」
- **Side Chat**：开一个不影响主线上下文的旁路对话，适合临时提问
- **Archive**：把暂时用不到的历史会话归档，保持主上下文干净

**Agent（代理层）**：负责执行任务的核心循环，支持 **Plan Mode（规划模式）**——先只读地研究代码、输出实施计划，等你确认后再动手改代码，避免 AI 直接乱改。

**Tools（工具层）**：Codex 能调用的具体能力：

- **Terminal**：执行 shell 命令（跑测试、构建等）
- **Git**：提交、查看 diff、管理分支
- **MCP**：接入外部工具生态（数据库、浏览器等）
- **Skill**：加载可复用的技能/流程文件

## AGENTS.md：项目级长期上下文

```
Codex
 ↓
读取 AGENTS.md
 ↓
知道项目规则
 ↓
执行任务
```

你可以把：

**AGENTS.md = 项目级长期上下文 / 操作规范**

它相当于告诉 Codex 这个项目的「规矩」：构建命令是什么、测试怎么跑、代码风格如何、哪些目录不能动。每次对话开始时自动加载，让 AI 不用每次都靠猜。

这和 Claude Code 的 `CLAUDE.md` 是同一个思路——用一份文件把项目的隐性知识显性化，让 AI 助手从第一天就懂你的项目。
