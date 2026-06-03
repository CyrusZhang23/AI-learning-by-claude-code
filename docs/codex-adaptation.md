# Codex CLI 适配参考

> **说明**：本课程以 Claude Code 为主线。这份文档给出 Codex CLI 用户的等价操作。
> ⚠️ Codex 的命令/配置项演进很快，下表的具体 flag 与 TOML 键名请以 [Codex 官方文档](https://github.com/openai/codex) 为准——概念映射是稳定的，实现细节可能已变化。

## 核心命名对照

| 概念 | Claude Code | Codex CLI |
|------|-------------|-----------|
| 配置文件格式 | JSON | TOML |
| 指令文件 | `CLAUDE.md` | `AGENTS.md` |
| 配置目录 | `.claude/` | `.codex/` |
| 用户配置 | `~/.claude/settings.json` | `~/.codex/config.toml` |
| 项目配置 | `.claude/settings.json` | `.codex/config.toml` |
| 单次执行 | `claude -p "..."` | `codex exec "..."` |
| 会话续接 | `claude --continue` / `--resume` | `codex resume --last` / `codex resume` |
| 管道输入 | `cat f \| claude -p "..."` | `cat f \| codex exec "..."` |
| 会话内 shell | `! git log` | `! git log` |
| 模型切换 | `/model` | `/model` |
| Skill 触发 | `/skill-name` | `$skill-name` |
| 自定义命令 | `.claude/commands/*.md` → `/name` | 用 skill 替代 |

## 逐层适配要点

- **层1 CLI**：高可移植。`claude` ↔ `codex`，`-p` ↔ `exec`，续接/管道/`!` 基本一致。
- **层2 Settings/Permissions**：高可移植。JSON → TOML；三层覆盖概念相同；权限上 Claude 用细粒度 specifier 规则，Codex 用预设沙箱模式（`read-only` / `workspace-write` / `danger-full-access`）+ 网络 allowlist。
- **层2 Hooks**：中低可移植。Codex 的 hook 事件数与 handler 类型少于 Claude（Claude 支持 command/http/mcp_tool/prompt/agent 多种 handler，Codex 以 shell command 为主）。Claude 独有的事件用外部 `fswatch`/`cron` 等替代。**具体事件清单请查 Codex 官方文档。**
- **层3 MCP**：高可移植。两者均全量支持 MCP。Codex 还能以 MCP server 模式被外部编排。
- **层4 多 Agent**：中可移植。单个 subagent 可移植；`isolation: "worktree"` 这种**声明式**隔离 Codex CLI 无直接等价，需手动 `git worktree`；Claude 的 Tasks 系统（TaskCreate/Update/List）Codex 无直接对应，用 markdown 状态文件 + 长任务模式替代。
- **层5 多 AI**：核心思路（文件/git 交接、MCP 直连）完全通用。把 Codex 当 MCP server 暴露，再用编排器调用，是这一层的常见做法。
- **层6 Agent SDK**：高可移植。把 Anthropic Agent SDK 换成 OpenAI Agents SDK，`Agent` / `Runner` / `handoff` 等概念一一对应。
- **层7 Skill**：高可移植。`SKILL.md` 格式两者通用，仅安装路径和触发语法（`/name` ↔ `$name`）不同。

## 不可直接移植、需替代方案的模块

| Claude Code 能力 | Codex 替代思路 |
|-----------------|---------------|
| `isolation: "worktree"`（声明式隔离） | 手动 `git worktree` 脚本封装 |
| Tasks 系统 | markdown 状态文件 + 长任务模式 |
| `CronCreate`（原生定时） | 外部 `crontab` + `codex exec` |
| 部分 hook 事件 / handler 类型 | 外部文件监听 / 仅 command handler |

> 用 Codex 学这门课时，遇到"不可移植"的步骤不要跳过——用上表的替代方案完成，反而能更深理解两个 harness 的设计差异。
