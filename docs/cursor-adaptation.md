# Cursor CLI 适配参考

> **说明**：本课程以 Claude Code 为主线（教学协议、课程范围、Personalization 协议、课程索引都在根目录 `CLAUDE.md`）。这份文档给出 **Cursor CLI（`cursor-agent`）/ Cursor IDE** 用户的等价操作。
> ⚠️ Cursor CLI 仍在快速演进（beta），下表的具体 flag、配置键名、文件路径请以 [Cursor 官方文档](https://cursor.com/docs/cli) 为准——**概念映射是稳定的，实现细节可能已变化**。

## 一个关键差异：Cursor 原生就读 `CLAUDE.md`

和 Codex 不同，Cursor CLI / IDE 在项目根目录会**同时读取 `AGENTS.md`、`CLAUDE.md` 和 `.cursor/rules/`**，都当作规则加载。这带来两个直接后果：

1. **好消息**：Cursor 用户开箱即得这门课的主指令——根目录 `CLAUDE.md` 里的「教学协议 / Personalization / 课程索引 / Coach notes」会被自动加载，不需要复制一份。
2. **要注意**：仓库里的 `AGENTS.md` 是 **Codex 入口**，里面的命令映射是给 Codex 的（`codex exec`、`.codex/config.toml` 等）。Cursor 也会读到它——**请忽略 `AGENTS.md` 的命令映射**，改用本文件的 Cursor 映射。`.cursor/rules/cursor-course.mdc` 已经把这条「忽略 Codex 映射」的指令固化为常驻规则。

> 一句话：**Cursor 把 `CLAUDE.md` 当主指令照常用，只把命令语法换成下面的 Cursor 版本即可。**

## 核心命名对照

| 概念 | Claude Code | Cursor CLI |
|------|-------------|------------|
| 进入交互模式 | `claude` | `cursor-agent` |
| 一次性 / headless | `claude -p "..."` | `cursor-agent -p "..."`（`--print`） |
| 结构化输出 | `claude -p --output-format json` | `cursor-agent -p --output-format json`（还有 `stream-json`） |
| 续接上一会话 | `claude --continue` | `cursor-agent --continue`（= `--resume=-1`）或 `cursor-agent resume` |
| 续接指定会话 | `claude --resume <id>` | `cursor-agent --resume <id>` |
| 列出历史会话 | `claude --resume`（picker） | `cursor-agent ls` |
| 管道输入 | `cat f \| claude -p "..."` | `cat f \| cursor-agent -p "..."` |
| 指定模型 | `claude --model ...` / `/model` | `cursor-agent --model ...` / `/model` |
| 指令文件 | `CLAUDE.md` | `AGENTS.md` / `.cursor/rules/*.mdc`（**Cursor 也读 `CLAUDE.md`**） |
| 项目/CLI 配置 | `.claude/settings.json` | `.cursor/cli-config.json` + `.cursor/rules/` |
| 权限 / 沙箱 | settings `permissions` | `.cursor/cli-config.json` 的 permissions + 审批模式 |
| Hooks | `.claude/settings.json` 的 `hooks` | `.cursor/hooks.json`（`version: 1`） |
| MCP | `.mcp.json` / settings | `.cursor/mcp.json`（`cursor-agent mcp` 管理） |
| 自定义斜杠命令 | `.claude/commands/*.md` → `/name` | `.cursor/commands/*.md` → `/name` |
| Skill | `.claude/skills/<n>/SKILL.md`，`/name` 触发 | `.cursor/skills/<n>/SKILL.md`，**按 description 自动匹配触发** |
| 规划/只读模式 | Plan / Ask 模式 | `/plan`、`/ask`，或 `Shift+Tab` 切换 |
| 后台 / 云端任务 | 后台任务 | 消息前加 `&` 交给 **Cloud Agent** 后台跑 |

> CI / 非交互场景下，Cursor 常需配合 `--force`（即 `--yolo`，跳过审批阻塞）和 `--trust`；MCP server 首次需批准，可用 `--approve-mcps`。

## 逐层适配要点（对应 `labs/layer-0…8`）

- **层0 摸底分班**：协议层，完全通用。本地化那一步（剪断 origin、解除 `workspace/` 忽略、清史重建）是纯 `git` 操作，Cursor 下命令完全相同。
- **层1 CLI 核心**：**高可移植**。`claude` ↔ `cursor-agent`，`-p` ↔ `-p`，`--continue` / `--resume` **连 flag 名都一样**；列历史用 `cursor-agent ls`。会话内执行 shell、管道输入概念一致。
- **层2 斜杠命令**：**高可移植，甚至更顺**。Codex 没有自定义斜杠命令，但 Cursor **原生支持** `.cursor/commands/*.md → /name`——本课已在 `.cursor/commands/` 放好 `/lab`、`/labs`、`/translate`，Cursor 用户开课体验和 Claude 一致。内置还有 `/plan`、`/ask`、`/resume`、`/create-rule`。
- **层3 Harness 配置**：**中高可移植**。
  - *Settings*：`.claude/settings.json` → `.cursor/cli-config.json`（CLI 行为/权限）+ `.cursor/rules/`（指令）。
  - *Hooks*：强对应——`.cursor/hooks.json`（`version: 1`）。事件名不同（如 `beforeShellExecution`、`afterFileEdit`、`beforeReadFile`、`beforeSubmitPrompt`、`stop` 等），command handler 通过 **stdin 收 JSON**，**退出码 2 表示阻断**（类比 Claude PreToolUse 的 block）。具体事件清单查官方文档。
  - *Permissions*：概念相同；Cursor 用审批模式 + allow/deny + 沙箱，粒度比 Claude 的 per-tool specifier 粗一些。
- **层4 MCP 集成**：**高可移植**。两者全量支持 MCP。Cursor 配置在 `.cursor/mcp.json`，用 `cursor-agent mcp` 管理，首次审批可用 `--approve-mcps`。自己写的 toy MCP server 直接复用，只换配置文件位置。
- **层5 多 Agent 协同**：**中可移植**。Cursor 有子代理 / Task 工具、`.cursor/worktrees`（worktree 隔离）、`&` 前缀的 Cloud Agent（后台/并行）。Claude 的 TaskCreate/Update/List 任务系统 → 用 Cursor 的 to-do 列表 + markdown 状态文件替代。
- **层6 多 AI · 多设备**：**高可移植（概念层）**。文件 / git 交接、MCP 直连的思路完全通用；"多设备"用 Cursor Cloud Agent（`&`）在云端跑长任务来模拟，本地终端不阻塞。
- **层7 Agent SDK**：**高可移植（换 SDK）**。把 Anthropic Agent SDK 换成 **Cursor SDK**（TypeScript `@cursor/sdk`，Python `cursor-sdk`）：`Agent.create` / `Agent.prompt` / `Agent.resume` / `run.stream` 等概念一一对应，可在脚本、CI、后端里编排 Cursor agent。
- **层8 Skill 工程**：**高可移植**。`SKILL.md` 格式两者通用，放 `.cursor/skills/<name>/SKILL.md`（用户级 `~/.cursor/skills/`）。**触发方式不同**：Claude 用 `/name` 显式触发，Cursor 按 SKILL.md 的 `description` **自动匹配上下文触发**——所以 description 要写得能精准命中使用场景。

## 需要注意 / 替代方案的模块

| Claude Code 能力 | Cursor 对应 / 替代 |
|-----------------|-------------------|
| `/name` 显式触发 skill | Cursor 按 `description` 自动匹配——把触发场景写进 description |
| Tasks 系统（TaskCreate/Update/List） | Cursor to-do 列表 + markdown 状态文件 |
| 声明式 `isolation: "worktree"` | Cursor `.cursor/worktrees` / Cloud Agent；或手动 `git worktree` |
| `CronCreate` 原生定时 | 外部 `crontab` + `cursor-agent -p "..."` |
| 后台并行长任务 | 消息前加 `&` → Cloud Agent |
| 多种 hook handler（http/mcp_tool/prompt/agent） | Cursor 以 command handler 为主，其余用外部脚本替代 |

> 用 Cursor 学这门课时，遇到"需替代方案"的步骤不要跳过——用上表方案完成，反而能更深理解两个 harness 的设计差异。

## 怎么用 Cursor 开这门课

```bash
git clone https://github.com/CyrusZhang23/AI-learning-by-claude-code.git
cd AI-learning-by-claude-code

cursor-agent          # 进入交互模式
# 然后在会话里：
/lab                  # 第一次自动进层0 摸底分班；之后从进度继续
/labs                 # 查看全部课程
/lab 1                # 直接开始第 1 层（已摸底过的话）
/translate            # 把当前这一层翻成英文
```

不想用斜杠命令也行——`.cursor/rules/cursor-course.mdc` 是常驻规则，Cursor 一进会话就知道这是教练课，直接说"我要上这门课，先做层 0 摸底"或"继续上课"即可。

> 维护约定（给课程开发者）：**不要把教学协议复制进本文件或 `.cursor/` 任何文件**。主指令永远只改根目录 `CLAUDE.md`。本文件只在 Cursor 命令映射 / 行为差异变化时同步。
