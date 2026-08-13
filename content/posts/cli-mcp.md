---
title: "CLI、Skill、MCP，到底有什么区别？"
date: 2026-08-14T00:07:00+08:00
draft: false
tags: ["AI", "MCP", "Agent", "CLI"]
---

最近在折腾 Agent 调用外部能力，发现 CLI、Skill、MCP 三个词经常被混在一起说。以高德地图为例，把 CLI 和 MCP 两条链路完整走一遍，区别就清楚了。

## 两段流程对比

以「查询周边 1km 咖啡店」为例，**高德 CLI** 的流程：

1. 用户输入自然语言：「查询周边 1km 咖啡店」
2. **SKILL 发现**：Agent 识别用户意图，匹配到对应的 SKILL
3. **CLI 命令构造**：LLM 参考 SKILL 中的流程，把自然语言参数转换为完整的 CLI 命令

   ```bash
   amap-gui searchPOI --keyword 星巴克 --city 北京 --pageSize 1
   ```

4. **执行 CLI 命令**：Agent 直接在终端执行构造好的命令
5. **请求后端接口**：CLI 工具解析命令参数，构造并发起 HTTP 请求调用高德 API
6. **结果反馈**：搜到的 POI 列表返回给 Agent，供其进一步处理

对比**高德 MCP（StreamableHTTP）**的流程：

1. 用户输入自然语言：「查询周边 1km 咖啡店」
2. **工具发现**：Agent 识别用户意图，匹配 MCP Server 暴露的工具描述
3. **MCP 请求构造**：LLM 参考工具的 `input_schema`，把自然语言参数转换为标准 JSON-RPC 消息
4. **工具调用请求**：MCP Client 通过 HTTP 向远程 MCP Server 发送 `tools/call` RPC 消息，包含请求参数
5. **请求后端接口**：MCP Server 解析 RPC 请求参数，构造并发起 HTTP 请求调用高德 API
6. **结果反馈**：MCP 返回的 POI 列表返回给 Agent，供其进一步处理

两条链路几乎一一对应，最大的区别在第三步：CLI 直接构造可执行命令，**省去了自然语言转 JSON-RPC 这一步消耗的 token**。

## 三者是什么关系

- **Skill = 如何做**：负责组织和指导，告诉 Agent 完成任务的具体流程
- **MCP = 用什么标准接口调用能力**：负责标准化地暴露工具
- **CLI = 具体执行能力的一种方式**：直接执行程序/命令

Skill 负责组织和指导，MCP 负责标准化地暴露工具，CLI 负责直接执行。它们处在不同的层面，并不冲突。

## MCP 首先是一种协议

MCP = Model Context Protocol，它本质上规定了一套 **AI 模型 / Agent 如何发现、理解、调用外部能力** 的通信协议。

严格来说，我们平时说「这个 MCP」，经常把几个不同的东西混在一起了。实际上可以拆成：

```
MCP
├── Protocol（协议）
├── MCP Server（服务端）
│     ├── Tools
│     ├── Resources
│     └── Prompts
└── MCP Client
      └── Agent / AI Application
```

**MCP 是协议，MCP Server 才是把一组能力按照这个协议暴露给 Agent 的东西。** 就像 HTTP 也是一种协议，我们不会把 HTTP 和具体的 Web 服务器混为一谈——但日常聊天时，「MCP」被直接当成了「MCP Server」用。

## 为什么 MCP 和 CLI 功能看起来重叠？

因为最终它们都在解决同一个问题：**Agent 怎么把自己的想法变成现实？**

以「创建一个 issue」为例，可以走 MCP：

```python
github.create_issue(...)
```

也可以走 CLI：

```bash
gh issue create ...
```

甚至可以走 HTTP API：

```bash
POST /repos/.../issues
```

三种方式殊途同归：MCP 胜在标准化、可发现（输入输出 schema 明确），CLI 胜在直接、省 token。选哪种，取决于场景。
