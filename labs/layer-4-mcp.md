# 层4 — MCP 集成

> 可移植性：**高**（Claude Code 与 Codex 均全量支持 MCP）

## 目标

理解 MCP（Model Context Protocol）——AI 与外部工具/数据之间的标准接口。学会把现成的 MCP server 接进来用，并**亲手写一个**最小 MCP server 暴露给 Claude Code。这是后面"多 AI 协作"的协议基础。

## 你将亲手产出

- `workspace/lab-03/my-mcp-server/` —— 你写的一个 toy MCP server（Python）
- 把它接进 Claude Code 的配置，并在会话里成功调用它的工具

## 前置检查

- [ ] Python 3.10+，能 `pip install`
- [ ] 理解层3 的权限概念（MCP 工具也受权限管控）
- [ ] 建好 `workspace/lab-03/` 工作区

## 分步教案

### 步骤 1 — 接一个现成的 MCP server
- **做什么**：选一个官方 server（如 filesystem 或 fetch），按其文档加进 MCP 配置（通常是 `command` + `args`）。重启会话后用 `/mcp` 查看已连接的 server 和它暴露的工具。
- **为什么**：先以"消费者"视角理解 MCP——一个 server 提供若干工具，Claude 通过协议发现并调用它们。
- **通过标准**：`/mcp` 里能看到该 server，且你能在会话里成功调用它的至少一个工具。

### 步骤 2 — 理解 MCP 的三类能力
- **做什么**：阅读并讲清 MCP 的三种暴露物——**tools**（可调用函数）、**resources**（可读数据）、**prompts**（预制提示）。对照步骤1 的 server 各属于哪类。
- **为什么**：知道 MCP 不只是"远程函数调用"，还能暴露数据和提示模板。
- **通过标准**：你能举例说明 tool / resource / prompt 各自的用途。

### 步骤 3 — 写一个最小 MCP server（核心）
- **做什么**：在 `my-mcp-server/` 用 Python MCP SDK 写一个 server，暴露 1–2 个工具，比如：`get_lab_progress()`（读 workspace 下各 lab 目录返回完成情况）、`roll_dice(n)`。先在本地用 SDK 自带的 inspector 跑通。
- **为什么**：以"生产者"视角真正理解 MCP——你定义工具的 schema，协议负责把它暴露给任何 MCP 客户端。
- **通过标准**：server 能独立启动，inspector 里能看到并调用你定义的工具。

### 步骤 4 — 把你的 server 接进 Claude Code
- **做什么**：把 `my-mcp-server` 加进 MCP 配置（指向你的启动命令），重启会话，用 `/mcp` 确认连接，然后让 Claude 调用你写的工具完成一个真实小任务（如"用 get_lab_progress 告诉我我学到第几层了"）。
- **为什么**：闭环——你造的能力现在成了 Claude 的工具。
- **通过标准**：Claude 能成功调用你的工具并用返回值回答问题。

### 步骤 5 — 权限作用域
- **做什么**：给你的 MCP 工具配权限规则（允许/询问/禁止），观察调用时的权限行为。
- **为什么**：MCP 工具和内置工具一样受 harness 权限管控——这是安全接入第三方能力的关键。
- **通过标准**：你能让某个 MCP 工具变成"调用前必须询问"。

## 完成标志

- 你接通了至少一个现成 MCP server
- 你**自己写的** MCP server 能被 Claude Code 调用
- 你能讲清 MCP 的 tools/resources/prompts 三类能力，以及 MCP 工具如何受权限管控

## 延伸（可选）

- 给你的 server 加一个 resource（如把某个 markdown 暴露为可读资源）
- 预习：层6 会把另一个 AI（Codex）当作 MCP server 来调用——这一层的 server 视角就是那时的基础
