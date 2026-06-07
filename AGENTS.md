# AI-learning-by-claude-code — Codex Entry Point

> 这门课以 Claude Code 为主线设计，**所有教学协议、课程范围、Personalization 协议、课程索引都在 `CLAUDE.md`**。本文件是 Codex 用户的薄壳入口，只放 Codex 必须知道的执行差异。**不双线维护教学协议**——协议更新只改 `CLAUDE.md`；Codex 能力映射更新到 `docs/codex-adaptation.md`。

---

## 给 Codex 的首要指令（每次会话开始执行）

1. **读取 `CLAUDE.md`** 并把里面的「Teaching protocol」「Personalization protocol」「Course scope」「Course index」「Coach notes」全部视为你的**主指令**。本文件不重复这些内容。
2. **读取 `workspace/progress.md`**（若存在），按其中的 track、skip list、节奏偏好定制讲解。不存在则按 `labs/layer-0-onboarding.md` 做摸底分班、生成 `workspace/progress.md`。
3. **读取 `docs/codex-adaptation.md` 的当前层映射**。教案文件（`labs/layer-*.md`，层 0–8）的概念、教学顺序和理解标准通用；遇到 Claude 专有命令、文件路径或完成标准时，必须换成 Codex 原生等价操作，不能要求 Codex 学员运行 Claude 命令。
4. **优先使用当前 Codex 原生能力**。Codex 已有项目配置、permissions、hooks、MCP、skills、plugins、subagents、Codex SDK、App worktrees 和 Automations；不要沿用旧适配里“Codex 没有这些能力”的判断。具体命令不确定时，先运行本机 `codex --help` / 对应子命令 `--help`，再查官方文档。

> 一句话：**先读 `CLAUDE.md`，再读 `progress.md`，再读当前层 Codex 映射，然后开始上课。**

## Codex 用户怎么开课

直接对 Codex 说：

> "我要上这门课，先做层 0 摸底。" 或 "继续上课。"

Codex 读取 `CLAUDE.md` + `workspace/progress.md` 后按协议带你做；需要某层时说“开始第 X 层”。不要把 Claude 的 `/lab` 当成 Codex 的课程入口。

## 命令语法差异（速查）

| 操作 | Claude Code | Codex |
|------|-------------|-------|
| 进入对话 | `claude` | `codex` |
| 一次性命令 | `claude -p "..."` | `codex exec "..."` |
| 续接会话 | `claude --continue` | `codex resume --last` |
| 指令文件 | `CLAUDE.md` | `AGENTS.md`（本文件） |
| 配置 | `.claude/settings.json` | `.codex/config.toml` |
| 结构化 headless 输出 | `--output-format json` | `codex exec --json`（JSONL 事件流） |
| MCP 管理 | `claude mcp ...` | `codex mcp ...` |
| Hooks 管理 | `/hooks` | `/hooks` |
| Skill 触发 | `/name` | `$name` |
| Plugin 管理 | `claude plugin ...` | `codex plugin ...` |

完整逐层映射、Codex 原生能力和真实差异见 **`docs/codex-adaptation.md`**。

## Codex 教学执行规则

- **保留原理，替换载体。** 例如层 3 的目标是掌握配置、权限和 hooks；Codex 学员应产出 `.codex/config.toml` / `.codex/hooks.json`，不是 `.claude/settings.json`。
- **完成标准也要翻译。** 教案写“用 `/mcp` 看见 server”时，Codex 可用 `/mcp` 或 `codex mcp list` 验证；写“Claude 能调用”时，改成“Codex 能调用”。
- **区分 CLI 与 App。** Codex CLI、Codex App、Codex Cloud 并非所有功能完全相同。Worktree 和 Automations 等涉及 App 的能力，要先确认学员使用的 surface；没有 App 时给 CLI/文件流水线替代路线。
- **不伪造一一对应。** Claude 的自定义 `.claude/commands/*.md`、特定 Task 工具或特定 hook payload 与 Codex 不完全相同。没有直接等价时，明确说明差异，再用 skill、plan/goal、脚本或对应 Codex机制完成同一学习目标。
- **版本敏感内容现场核验。** 本仓库适配文档给稳定映射；具体 flag、TOML key、hook payload 和可用 slash command，以学员本机版本和 Codex 官方文档为准。

## 维护约定（给课程开发者）

- **不要把教学协议复制到本文件**。如果某天你想"在 AGENTS.md 里改一下教学协议"，停——去改 `CLAUDE.md`。
- 本文件只在以下情况修改：(a) Codex 开课入口变化；(b) 新增关键 Claude/Codex 命令映射；(c) Codex 教练执行规则变化；(d) `docs/codex-adaptation.md` 路径变化。
- 如果你发现 Codex 表现与 Claude 不一致，请先确认 Codex 是否真的读了 `CLAUDE.md`——大部分行为差异是"没读主指令"而不是"协议不同"。
