# Codex 适配参考

> **定位**：本课程以 Claude Code 为主线，教学协议和课程索引仍以根目录 `CLAUDE.md` 为唯一来源。本文件只负责把层 0–8 的 Claude 实操翻译成 Codex 原生操作。
>
> **核验基线**：2026-06-07，Codex CLI `0.137.0`。Codex 演进很快；具体 flag、TOML key、hook payload 和 surface 可用性，以学员本机 `codex --help`、对应子命令 `--help` 和 [Codex 官方文档](https://developers.openai.com/codex/) 为准。

## 适配原则

1. **教学目标不变，执行载体替换。** 教案写 `.claude/settings.json` 时，Codex 路线使用 `.codex/config.toml` 或 `.codex/hooks.json`；教案写“Claude 能调用”时，完成标准改成“Codex 能调用”。
2. **优先原生能力，不使用过时替代方案。** 当前 Codex 原生支持项目配置、permissions、hooks、MCP、skills、plugins、subagents、Codex SDK、App worktrees 和 Automations。
3. **区分 surface。** CLI、App、IDE 和 Cloud 的能力不完全一致。开始需要 worktree、Automations 或 Cloud 的步骤前，先确认学员正在使用哪个 surface。
4. **没有直接等价时明确说差异。** 不把 Claude 专有概念硬说成 Codex 一一对应；用 Codex 的 skill、plan/goal、脚本或文件流水线完成同一学习目标。
5. **现场核验版本敏感事实。** 每层全景扫描优先运行本机 `codex --help`、`codex <subcommand> --help` 和交互式 slash command 列表。

## 核心命名与命令对照

| 概念 | Claude Code | Codex |
|------|-------------|-------|
| 交互模式 | `claude` | `codex` |
| 单次执行 | `claude -p "..."` | `codex exec "..."` |
| 管道输入 | `cat f \| claude -p "..."` | `cat f \| codex exec "..."` |
| 续接最近交互会话 | `claude --continue` | `codex resume --last` |
| 续接最近 headless 会话 | `claude --continue -p "..."` | `codex exec resume --last "..."` |
| 结构化输出 | `--output-format json` | `codex exec --json`，输出 JSONL 事件流 |
| 保存最终回答 | shell 重定向 | `codex exec -o <file> "..."` |
| 指令文件 | `CLAUDE.md` | `AGENTS.md`；目录树内更近的文件作用域更窄 |
| 用户配置 | `~/.claude/settings.json` | `~/.codex/config.toml` |
| 项目配置 | `.claude/settings.json` | `<repo>/.codex/config.toml`，项目受信任后加载 |
| 一次性配置覆盖 | CLI flags | `-c key=value` / 专用 flags |
| 沙箱/权限 | permissions allow/ask/deny | permission profiles + approval policy + rules |
| Hooks | settings 内 hooks | `.codex/hooks.json` 或 `.codex/config.toml` 内 `[hooks]` |
| Hooks 查看 | `/hooks` | `/hooks` |
| MCP 管理 | `claude mcp ...` | `codex mcp add/list/get/remove/login/logout` |
| Codex 作为 MCP server | 不适用 | `codex mcp-server` |
| Skill 触发 | `/name` / 自动匹配 | `$name`、`/skills`、自然语言匹配 |
| Plugin 管理 | `claude plugin ...` | `codex plugin ...` |
| 非交互代码审查 | prompt / command | `codex review` 或 `codex exec review` |
| 子 agent | Agent 工具 / 自定义 agent | 原生 subagents / custom agents |
| SDK | Claude Agent SDK | Codex SDK：`@openai/codex-sdk` 或 `openai-codex` |

## 逐层执行映射

### 层 0：摸底与环境

- 环境检查把 `claude --version` 换成 `codex --version`，并用 `codex login` 检查或完成登录。
- `AGENTS.md` 是 Codex 的课程入口，但主教学协议仍来自 `CLAUDE.md`。
- 本地化、Git、`workspace/progress.md` 和 A/B/C 路线完全通用。

### 层 1：CLI 核心与会话

- 交互/headless：`claude` ↔ `codex`，`claude -p` ↔ `codex exec`。
- Claude 的单个 JSON 结果改为 Codex 的 `codex exec --json` JSONL 事件流；若只要最终文本，用 `-o <file>`。
- 会话续接：交互用 `codex resume --last`；headless 用 `codex exec resume --last "继续任务"`。
- Claude 的 `#` 速记和 `/memory` 不按原样照搬。Codex 长期项目规则写进 `AGENTS.md`；若本机启用了 memories，可用 `/memories` 观察，但不要把实验建立在实验性功能上。
- 收尾运行 `codex --help`、`codex exec --help`、`codex resume --help` 建立全景。

### 层 2：斜杠命令、Skills 与可复用工作流

- 内置斜杠命令以学员本机交互界面为准；重点观察 `/skills`、`/hooks`、`/mcp`、`/model` 等入口。
- Claude 的 `.claude/commands/*.md` 没有应被假装成完全相同的 Codex 文件机制。本层 Codex 核心产出改为一个可复用 skill，并用 `$skill-name`、`/skills` 和自然语言分别验证触发。
- Claude 自定义命令里的 `$ARGUMENTS`、位置参数和 shell 插值，Codex 路线改为理解 skill 的触发契约、说明正文与内嵌脚本之间的分工。
- 完成标准：能区分 Codex 内置 slash command、skill 和 plugin 提供的能力；能解释为什么固定流程更适合封装成 skill。

### 层 3：Harness 配置、权限与 Hooks

- 配置层：用户级 `~/.codex/config.toml`；项目级和目录级 `.codex/config.toml`。Codex 会从项目根到当前目录加载配置，更近目录的配置优先；项目配置只在 trusted project 中加载。
- 一次性覆盖使用 `codex -c key=value` 或专用 flags。不要把 Claude 的 `settings.local.json` 生搬成 Codex 文件。
- 权限实验使用内置 permission profiles（如只读、workspace write）或自定义 profile，并观察 filesystem/network deny 规则与 approval policy。
- Hooks 原生可用：放在 `.codex/hooks.json` 或 `.codex/config.toml`，使用 `/hooks` 检查来源、review 和 trust。Codex 支持包括 `PreToolUse`、`PermissionRequest`、`PostToolUse`、`UserPromptSubmit`、`Stop`、`SubagentStart/Stop` 等生命周期事件。
- Hook payload、阻断语义和 trust 流程必须按 Codex 官方文档与本机版本验证，不直接复制 Claude JSON 字段或退出码结论。
- 完成标准：项目配置生效、权限边界生效、至少一个审计 hook 与一个拦截/验证 hook 生效。

### 层 4：MCP

- MCP server 本身完全复用；FastMCP toy server 不需要为 Codex 重写。
- 接入示例：

  ```bash
  codex mcp add my-server -- uv run --directory <绝对路径>/my-mcp-server server.py
  codex mcp list
  codex mcp get my-server
  ```

- 交互界面可用 `/mcp` 查看状态；具体工具审批策略在 Codex config 中配置。
- 完成标准改成：Codex 能发现并调用 toy MCP server，学员能审查工具 schema、边界和权限。

### 层 5：多 Agent 协同

- 当前 Codex 原生支持并行 subagents 和 custom agents；不要再写成“Codex 只有单个 subagent”。
- 探索、实现、独立只读 reviewer 三类职责完全可复刻。权限边界用 agent 配置和当前会话 permission profile 控制。
- Worktree 要区分 surface：
  - Codex App 支持 Codex-managed worktrees 和 Local/Worktree handoff。
  - 只使用 CLI 时，可用手动 `git worktree` 或独立工作目录完成同一隔离实验。
- Claude Tasks 不要求逐字复刻。Codex 路线用当前会话提供的 plan/goal 能力；若 surface 没有对应能力，再用 markdown 状态文件。
- 完成标准：能派发并行 subagents、定义专长 agent、建立独立 reviewer 闸门，并完成一次隔离修改。

### 层 6：多 AI 与多设备协作

- 文件交接、机器可读哨兵、独立只读裁判和打回重试完全通用。
- Codex 可作为流水线中的探索方、实现方或 reviewer：

  ```bash
  codex exec -s read-only "审查实现并输出 VERDICT:PASS 或 VERDICT:FAIL"
  codex exec -s workspace-write "按 findings.md 修改并验证"
  ```

- Codex 作为 MCP server 使用 `codex mcp-server`，供 Claude/Cursor 或其他 MCP client 调用。
- Codex App 的 Automations 可承担定时任务；Codex Cloud/App 可承担远程执行。只有 CLI 时，继续使用脚本、cron 或 CI 作为替代。
- 完成标准：至少跑通文件流水线；有合适 client 时再跑 MCP 直连；能解释触发、执行、回流三种职责。

### 层 7：Codex SDK

- Codex 主路线使用 **Codex SDK**，不是 OpenAI Agents SDK：
  - TypeScript：`npm install @openai/codex-sdk`
  - Python：`pip install openai-codex`
- Codex SDK 用于程序化控制本地 Codex thread/app-server，适合复刻“SDK orchestrator → Codex runtime → 模型/API”的分层。
- OpenAI Agents SDK 是另一条“直接构建 API agent 应用”的路线，可作为对照加餐，但不应替代 Codex SDK。
- 完成标准：能启动/续接 Codex thread，观察事件或结果，并实现一个小型编排流程。

### 层 8：Skill 与 Plugin

- `SKILL.md` 的核心思想通用：description 是触发契约，正文按需加载，确定性工作交给脚本。
- Codex 使用 `$skill-name`、`/skills` 和自然语言验证 skill；具体 skill 安装位置与发现规则以当前官方文档和本机 `/skills` 为准。
- Codex plugin 的必需入口是 `.codex-plugin/plugin.json`，可打包 skills、hooks、MCP servers、apps/connectors 和 assets。
- 使用 `codex plugin marketplace ...`、`codex plugin add ...`、`codex plugin remove ...` 完成分发实验；先运行对应 `--help` 获取本机准确参数。
- 完成标准：在干净环境中安装 plugin，能触发其中的 skill，并能解释 plugin 与 skill 的职责差异。

## 当前真正需要区别处理的能力

| Claude 教案内容 | Codex 处理方式 |
|----------------|---------------|
| `.claude/commands/*.md` 自定义斜杠命令 | 用 skill 封装同一工作流；不要伪装成完全等价机制 |
| `settings.local.json` | 使用目录级 `.codex/config.toml`、profile 或 CLI `-c` 覆盖 |
| Claude 特定 permissions specifier | 使用 Codex permission profiles、approval policy、rules；语法不直接复制 |
| Claude hook payload/阻断约定 | 使用 Codex 原生 hooks，但按 Codex 文档验证 payload、输出与阻断语义 |
| Claude TaskCreate/Update/List | 优先用当前 Codex surface 的 plan/goal；否则 markdown 状态文件 |
| `isolation: "worktree"` | Codex App 用 managed worktree；纯 CLI 用手动 `git worktree` |
| `CronCreate` / RemoteTrigger / PushNotification | Codex App Automations / Cloud / thread automation；纯 CLI 用 cron/CI/脚本 |
| Claude Agent SDK | Codex 路线使用 Codex SDK；OpenAI Agents SDK 只作 API agent 对照 |
| `.claude-plugin/plugin.json` | Codex 使用 `.codex-plugin/plugin.json` |

## 官方核验入口

- [Codex 文档首页](https://developers.openai.com/codex/)
- [Advanced configuration](https://developers.openai.com/codex/config-advanced/)
- [Permissions](https://developers.openai.com/codex/permissions/)
- [Hooks](https://developers.openai.com/codex/hooks/)
- [MCP](https://developers.openai.com/codex/mcp/)
- [Subagents](https://developers.openai.com/codex/subagents/)
- [Codex SDK](https://developers.openai.com/codex/sdk/)
- [Build plugins](https://developers.openai.com/codex/plugins/build/)
- [Automations](https://developers.openai.com/codex/app/automations/)
- [Worktrees](https://developers.openai.com/codex/app/worktrees/)

> 用 Codex 学这门课时，不跳过 Claude 专有步骤，而是先说清差异，再用 Codex 原生能力完成同一个学习目标。这样学到的是 harness 设计，而不是死记某个产品的命令。
