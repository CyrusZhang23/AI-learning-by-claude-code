# 层4 — MCP 集成

> 可移植性：**高**（Claude Code 与 Codex 均全量支持 MCP）

> 🎯 **本层一句话**：MCP 是标准协议 —— 协议怎么跑 = Anthropic 的事（不用 care）；工具暴露什么 / 边界 / 权限给哪一档 = 你必须 care；代码怎么写 = AI 写、你审。

## 开篇定调：你只需要 care 一件事

学这层前先把心态摆正，否则会被一堆命令淹没。

**MCP 是一个标准协议**，让"提供工具的程序（server）"和"使用工具的 AI（client，如 Claude/Codex）"用统一格式对话：

```
  ┌──────────┐      MCP 协议（一问一答的 JSON）      ┌────────────┐
  │  Claude   │  ←──────────────────────────────→  │  你的 server │
  │ (client)  │  ① 你有啥工具? ② 帮我调X ③ 给结果   │ (提供工具)   │
  └──────────┘                                      └────────────┘
```

server 一启动，两边发生三个回合：**① 握手发现工具清单 → ② Claude 请求调用某工具 → ③ server 执行并回传结果**。

**关键：这三个回合的协议对话、JSON 收发、schema 生成、传输层、inspector 网页怎么拉起来、Python 进程怎么跑、Claude 和 Codex 怎么通信——全是 SDK（`FastMCP`）和 Anthropic 替你做的，你一行都不用碰。那是你花钱买的服务。**

**你唯一要 care 的，是 `@mcp.tool()` 装饰的函数里写什么**：

```python
@mcp.tool()                      # ← 这个装饰器以上的一切，不用管
def roll_dice(n: int) -> int:    # ← 你只负责这个函数：
    """掷一个 n 面骰子。"""        #    叫什么、要什么参数、干什么活
    return random.randint(1, n)
```

所以本层你的真实能力目标不是"从零手写 MCP server"，而是：

| 你的角色 | 具体职责 |
|---|---|
| **决策者** | 决定暴露哪些工具、每个工具的边界、权限给 allow/ask/deny 哪一档 |
| **审查者** | 看懂 AI 写出来的 server，能判断它对不对、危不危险（如：漏了类型标注→AI 调用会崩；写了 `run_shell` 还默认放行→等于敞开 shell） |
| **指挥者** | 指挥 AI 写工具函数，自己不手敲实现 |

下面的步骤会带你走一遍完整流程，但记住：**协议怎么跑 = Anthropic 的事（不用 care）；工具写什么 = 你的事（必须 care）；代码怎么写 = AI 的事（你审）。**

## 目标

理解 MCP（Model Context Protocol）——AI 与外部工具/数据之间的标准接口。学会把现成的 MCP server 接进来用，并指挥 AI 写一个最小 MCP server 暴露给 Claude Code（你审查、决策，不手写）。这是后面"多 AI 协作"的协议基础。

## 你将产出（指挥 AI 写，你审查决策）

- `workspace/lab-04/my-mcp-server/` —— 一个 toy MCP server（Python，AI 写、你定工具）
- 把它接进 Claude Code 的配置，并在会话里成功调用它的工具
- **能力产出**：看懂这个 server、能判断工具边界与权限档位

## 前置检查

- [ ] **Python 环境管理统一用 `uv`**（本课程所有 Python 实验的约定）。先探明系统现状：
  ```bash
  uv --version          # 有 → 直接用
  conda --version       # 没 uv 但有 anaconda/miniconda → 也可，但本课示例以 uv 为准
  ```
  两者都没有 → 推荐装 uv（比 pip/conda 快、自带虚拟环境与 Python 版本管理）：
  ```bash
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```
  装完重开终端，`uv --version` 能出版本即可。
- [ ] Python 3.10+（uv 会按需自动下载，不用手动装）
- [ ] 理解层3 的权限概念（MCP 工具也受权限管控）
- [ ] 建好 `workspace/lab-04/` 工作区

> **为什么统一 uv**：MCP server 有依赖（`mcp` SDK），裸 `pip install` 会污染全局环境。`uv` 用项目级虚拟环境隔离，`uv run server.py` 自动按 `pyproject.toml` 拉依赖再跑——干净、可复现。后续步骤的命令都以 `uv` 给出。

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
- **做什么**：用 `uv` 初始化项目并写一个 server，暴露 1–2 个工具，比如 `roll_dice(n)`、`get_lab_progress()`（读 workspace 下各 lab 目录返回完成情况）。
  ```bash
  cd workspace/lab-04
  uv init my-mcp-server && cd my-mcp-server
  uv add "mcp[cli]"          # 装 MCP Python SDK + inspector
  ```
  写好 `server.py` 后用 inspector 本地跑通：
  ```bash
  uv run mcp dev server.py   # 打开 inspector，可视化调用工具
  ```
- **▶ 让学员先预测**：教练写好 `roll_dice(n: int) -> int` 后、跑 inspector 之前，问学员两题——① "Claude 怎么知道这个工具要一个**整数**参数？我哪儿声明的？"（答案：`: int` 类型标注，SDK 据此自动生成 JSON schema）② "如果我把 `: int` 删掉、参数不写类型，会怎样？"（答案：schema 缺类型信息，Claude 调用时容易传错 / 崩——这正是审查者要抓的）。让学员答完再用 inspector 验证。
- **为什么**：以"生产者"视角真正理解 MCP——你定义工具的 schema，协议负责把它暴露给任何 MCP 客户端。**关键机制**：schema 不是你手写的，是 SDK 从**类型标注**生成、描述从**docstring** 生成——所以函数签名和 docstring 就是工具的对外契约。
- **通过标准**：server 能独立启动，inspector 里能看到并调用你定义的工具；学员能指出"是 `: int` 和 docstring 决定了 Claude 看到的工具长什么样"。

### 步骤 4 — 把你的 server 接进 Claude Code
- **做什么**：把 `my-mcp-server` 加进 MCP 配置，启动命令用 `uv run`（自动带上虚拟环境，不依赖全局 Python）：
  ```bash
  claude mcp add my-server -- uv run --directory <绝对路径>/my-mcp-server server.py
  ```
  重启会话，用 `/mcp` 确认连接，然后让 Claude 调用你写的工具完成一个真实小任务（如"用 get_lab_progress 告诉我我学到第几层了"）。
- **为什么**：闭环——你造的能力现在成了 Claude 的工具。
- **通过标准**：Claude 能成功调用你的工具并用返回值回答问题。

### 步骤 5 — 权限作用域
- **做什么**：给你的 MCP 工具配权限规则（允许/询问/禁止），观察调用时的权限行为。
- **为什么**：MCP 工具和内置工具一样受 harness 权限管控——这是安全接入第三方能力的关键。
- **通过标准**：你能让某个 MCP 工具变成"调用前必须询问"。

### 步骤 6 — 全景扫描（收尾）[核心]
- **做什么**：对照 MCP 的**三类暴露物**（tools / resources / prompts），问学员"我们这层动手写的是哪类？另两类各是干啥的、什么时候用？"——本层主要做了 tools，resources/prompts 只读到概念。再对照 `claude mcp` 子命令全集（`add` / `list` / `remove` / `get`），明确还有哪些没用过。
- **为什么**：建立"MCP 三类能力 + mcp 子命令"的索引感，知道 tools 只是其中一类。

> **🧪 留个回归夹具**：把 `my-mcp-server/`（含 `roll_dice`、`get_lab_progress`）和接入命令 `claude mcp add lab-server -- uv run --directory <path> server.py` 留作夹具。升级 SDK 或改教案后，重跑 `uv run mcp dev server.py` 就能确认工具仍能被发现和调用。

## 完成标志

- 你接通了至少一个现成 MCP server
- 你**自己写的** MCP server 能被 Claude Code 调用
- 你能讲清 MCP 的 tools/resources/prompts 三类能力，以及 MCP 工具如何受权限管控
- 🎯 能复述本层那句话：协议=Anthropic 的事不用 care，工具写什么/边界/权限=你必须 care，代码怎么写=AI 写你审

## 延伸（可选）

- 给你的 server 加一个 resource（如把某个 markdown 暴露为可读资源）
- 预习：层6 会把另一个 AI（Codex）当作 MCP server 来调用——这一层的 server 视角就是那时的基础
