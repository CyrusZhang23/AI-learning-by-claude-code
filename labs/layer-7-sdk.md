# 层7 — Agent SDK

> 可移植性：**高**（概念通用；Codex 换成 OpenAI Agents SDK，见 `docs/codex-adaptation.md`）

> 🎯 **本层一句话**：三层洋葱 —— 你写的 SDK orchestrator 坐在 `claude` CLI 之上，CLI 又坐在原始 API 之上；CLI 就是那层把**权限 / 限流 / 上下文压缩 / hook / 渲染**都包好的 harness，裸用 SDK 这些得自己处理。

## 目标

跳出 CLI，用 **Agent SDK** 自己写一个多 agent 编排器，从而真正理解前面几层的 CLI harness 在底层是怎么运转的——工具调用循环、子 agent、handoff、上下文管理。

## 你将亲手产出

- `workspace/lab-07/` —— 一个能跑的小型 orchestrator（用 SDK 实现层5 的"探索→实现"两段式编排）

## 前置检查

> **这一层默认走订阅态，不需要单独 API key。** 各家官方 SDK 本质都是"包装自家 CLI 的库"——在底层 spawn 一个 CLI 子进程，复用你已经登录的订阅态。所以"裸调 API、单独付费 key"在本课里只作对照，不作默认门槛。

- [ ] **订阅态（默认）**：`claude` CLI 已登录（`claude --version` 能跑即可）。SDK 用 `claude-agent-sdk`（Python，建议 `uv add claude-agent-sdk`）。
  - Codex 学员对等：`codex` CLI 已 `codex login`（ChatGPT 登录态），SDK 用 `@openai/codex-sdk`（TS），headless 入口 `codex exec`。
- [ ] **API key（可选/进阶）**：若要走裸 Anthropic Messages API / OpenAI Agents SDK 做对比，再准备对应 key 并设环境变量。
- [ ] Python（走 `claude-agent-sdk`，本课主线）或 Node（走 TS SDK）环境
- [ ] 完成层5（理解多 agent 编排的概念）

> **三层洋葱**（贯穿本层，步骤4 会回望）：你写的 **SDK orchestrator** 坐在 **`claude` CLI** 之上，CLI 又坐在 **原始 Anthropic API** 之上。订阅态 SDK 之所以不用 key，正是因为它借道中间那层 CLI 的登录。

## 分步教案

### 步骤 1 — 最小 agent
- **做什么**：用 Agent SDK 起一个最小 agent，给它一个简单任务跑通（如"返回今天的待办清单"）。开启 prompt caching。
- **▶ 让学员先预测**：跑之前问——"`query()` 调完，你觉得它直接返回一个字符串答案，还是别的东西？" 多数人答"返回答案字符串"。**揭晓：返回一个异步消息流**，里面两类事件交错：语义事件（Assistant/Result）+ harness 事件（Hook/System/RateLimit）。再追问："消息流里居然有 `HookEventMessage`——这说明什么？"（答案：你在层 3 配的 hook 在底层 CLI 真开火了，**证明 hook 引擎是 CLI 的，SDK 是白嫖 CLI**——正好印证三层洋葱。）
- **为什么**：先把 SDK 的"模型调用 + 工具循环"骨架跑通，理解最小闭环；并通过消息流看见"SDK 坐在 CLI 之上"。
- **通过标准**：一个 SDK agent 能接收输入、调用模型、返回结果；学员能说出"返回的是消息流不是字符串，且流里能看到 hook 事件"。

### 步骤 2 — 给 agent 加工具
- **做什么**：定义 1–2 个本地工具（如读文件、跑 shell），注册给 agent，让它在完成任务时自己决定调用工具。
- **为什么**：理解"工具调用循环"——模型决定调哪个工具、SDK 执行、结果回灌、再决策，直到完成。这正是 CLI 里发生的事。
- **通过标准**：agent 能自主调用你的工具完成一个需要外部信息的任务。

### 步骤 3 — 多 agent + handoff
- **做什么**：建两个 agent：`explorer`（只读，产出分析）和 `implementor`（据分析改代码），让 implementor 能 handoff 给 explorer。给一个真实任务（优化 workspace 里的脚本）。
- **▶ 让学员先预测**：这个流水线只有 explorer + implementor，没有独立 reviewer。问学员——"还记得层 6 那条铁律吗？这个缺了只读评审闸门的 SDK 流水线，implementor 自报'行为等价'时，靠得住吗？" **揭晓**：实测 implementor 把 `[3,1,5]` 排序去重改成 `[1,5,3]`（破坏了顺序不变量）却自报"等价"——没有独立只读评审就漏网了，**坐实层 6 的铁律在 SDK 里同样成立**。
- **为什么**：handoff 是多 agent 协作的核心机制——一个 agent 把控制权和上下文交给另一个。对应层5 的子 agent 派发。**注意**：SDK 多 agent = 主 agent 用 `Agent` 工具派发子 agent（本机工具名是 `Agent` 不是 `Task`）；`tools=` 字段就是权限边界（explorer 只读，物理上改不了文件）。
- **通过标准**：任务在两个 agent 间正确流转，最终产出改好的代码；学员能指出"缺独立 reviewer 闸门是这条流水线的隐患"。

### 步骤 4 — 对照 CLI harness
- **做什么**：把你这个 SDK orchestrator 和层5 用 CLI 做的同类编排对比，列出：CLI 帮你自动做了哪些事（权限弹窗、上下文压缩、子 agent 隔离），SDK 里需要你自己处理哪些。
- **为什么**：这一步是整门课的"原理回望"——理解 CLI 是 SDK 之上的一层 harness。
- **通过标准**：你能列出至少 3 件"CLI 自动做、SDK 要手动做"的事。

### 步骤 5 — 全景扫描（收尾）[核心]
- **做什么**：把步骤 4 列的"CLI 自动做 vs 裸 SDK 自理"扩成一张总账，逐项问学员归到哪边：① 限流（CLI 优雅提示 / SDK 抛 Exception）② 权限（CLI=settings 三层+真人弹窗 / SDK headless 代码写死 `permission_mode`+`allowed_tools`）③ hook 引擎 ④ 历史自动压缩 ⑤ 消息渲染 ⑥ 子 agent 隔离（这条 SDK 也给）⑦ 等价闸门（谁都不给，得自装）。明确"CLI 这层到底替你包了什么"。
- **为什么**：这是整门课的原理收口——把前 6 层用过的 harness 能力，逐一对应到"SDK 之上那层 CLI 做了什么"。

> **🧪 留个回归夹具**：`workspace/lab-07/` 的 uv 项目（`step1_minimal.py` / `step2_tool.py` / `step3_orchestrator.py` + `slow_dedup.py`）就是夹具。⚠️ 订阅态共享 session 配额，多 agent 一跑（编排者+2子agent）烧得快、可能触限流——重跑夹具时留意这点。

## 完成标志

- 你有一个能跑的多 agent SDK orchestrator
- 你能解释"工具调用循环"和"handoff"两个机制
- 你能讲清 CLI harness 相比裸用 SDK 多提供了什么
- 🎯 能复述本层那句话：三层洋葱，CLI 是把权限/限流/压缩/hook/渲染包好的 harness 层

## 延伸（可选）

- 给 orchestrator 加上下文管理（长任务自动压缩历史）
- 用 SDK 复刻层6 的多工具流水线，通过 MCP 调用外部 AI
- 给步骤 3 的流水线补一个只读 reviewer 子 agent，复刻层 6"打回→修→复审"闭环（补上那条漏掉的等价闸门）
