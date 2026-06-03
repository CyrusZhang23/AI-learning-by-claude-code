# 层6 — Agent SDK

> 可移植性：**高**（概念通用；Codex 换成 OpenAI Agents SDK，见 `docs/codex-adaptation.md`）

## 目标

跳出 CLI，用 **Agent SDK** 自己写一个多 agent 编排器，从而真正理解前面几层的 CLI harness 在底层是怎么运转的——工具调用循环、子 agent、handoff、上下文管理。

## 你将亲手产出

- `workspace/lab-06/` —— 一个能跑的小型 orchestrator（用 SDK 实现层4 的"探索→实现"两段式编排）

## 前置检查

- [ ] 有 Anthropic API key（或对应 SDK 的 key），设好环境变量
- [ ] Python 或 Node 环境（按你选的 SDK）
- [ ] 完成层4（理解多 agent 编排的概念）

## 分步教案

### 步骤 1 — 最小 agent
- **做什么**：用 Agent SDK 起一个最小 agent，给它一个简单任务跑通（如"返回今天的待办清单"）。开启 prompt caching。
- **为什么**：先把 SDK 的"模型调用 + 工具循环"骨架跑通，理解最小闭环。
- **通过标准**：一个 SDK agent 能接收输入、调用模型、返回结果。

### 步骤 2 — 给 agent 加工具
- **做什么**：定义 1–2 个本地工具（如读文件、跑 shell），注册给 agent，让它在完成任务时自己决定调用工具。
- **为什么**：理解"工具调用循环"——模型决定调哪个工具、SDK 执行、结果回灌、再决策，直到完成。这正是 CLI 里发生的事。
- **通过标准**：agent 能自主调用你的工具完成一个需要外部信息的任务。

### 步骤 3 — 多 agent + handoff
- **做什么**：建两个 agent：`explorer`（只读，产出分析）和 `implementor`（据分析改代码），让 implementor 能 handoff 给 explorer。给一个真实任务（优化 workspace 里的脚本）。
- **为什么**：handoff 是多 agent 协作的核心机制——一个 agent 把控制权和上下文交给另一个。对应层4 的子 agent 派发。
- **通过标准**：任务在两个 agent 间正确流转，最终产出改好的代码。

### 步骤 4 — 对照 CLI harness
- **做什么**：把你这个 SDK orchestrator 和层4 用 CLI 做的同类编排对比，列出：CLI 帮你自动做了哪些事（权限弹窗、上下文压缩、子 agent 隔离），SDK 里需要你自己处理哪些。
- **为什么**：这一步是整门课的"原理回望"——理解 CLI 是 SDK 之上的一层 harness。
- **通过标准**：你能列出至少 3 件"CLI 自动做、SDK 要手动做"的事。

## 完成标志

- 你有一个能跑的多 agent SDK orchestrator
- 你能解释"工具调用循环"和"handoff"两个机制
- 你能讲清 CLI harness 相比裸用 SDK 多提供了什么

## 延伸（可选）

- 给 orchestrator 加上下文管理（长任务自动压缩历史）
- 用 SDK 复刻层5 的多工具流水线，通过 MCP 调用外部 AI
