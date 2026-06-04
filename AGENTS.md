# AI-learning-by-claude-code — Codex Entry Point

> 这门课以 Claude Code 为主线设计，**所有教学协议、课程范围、Personalization 协议、课程索引都在 `CLAUDE.md`**。本文件 = Codex 用户的薄壳入口，只放 Codex 必须知道的差异。**不双线维护**——内容更新只改 `CLAUDE.md`，本文件只在协议入口或命令映射变化时同步。

---

## 给 Codex 的首要指令（每次会话开始执行）

1. **读取 `CLAUDE.md`** 并把里面的「Teaching protocol」「Personalization protocol」「Course scope」「Course index」「Coach notes」全部视为你的**主指令**。本文件不重复这些内容。
2. **读取 `workspace/progress.md`**（若存在），按其中的 track、skip list、节奏偏好定制讲解。不存在则按 `labs/layer-0-onboarding.md` 做摸底分班、生成 `workspace/progress.md`。
3. 教案文件（`labs/layer-*.md`）的语言、结构、动手要求两端通用——遇到 Claude 专有命令时，按下方「命令语法差异」翻译，按 `docs/codex-adaptation.md` 处理不可移植模块。

> 一句话：**先读 CLAUDE.md，再读 progress.md，再开始上课**。本文件只为命令翻译服务。

## Codex 用户怎么开课

Codex 没有 `/lab` 这类自定义斜杠命令。直接对 Codex 说：

> "我要上这门课，先做层 0 摸底。" 或 "继续上课。"

Codex 读取 `CLAUDE.md` + `workspace/progress.md` 后按协议带你做；需要某层时说"开始第 X 层"。

## 命令语法差异（速查）

| 操作 | Claude Code | Codex |
|------|-------------|-------|
| 进入对话 | `claude` | `codex` |
| 一次性命令 | `claude -p "..."` | `codex exec "..."` |
| 续接会话 | `claude --continue` | `codex resume --last` |
| 指令文件 | `CLAUDE.md` | `AGENTS.md`（本文件） |
| 配置 | `.claude/settings.json` | `.codex/config.toml` |
| Skill 触发 | `/name` | `$name` |

完整对照和"不可移植模块的替代方案"（worktree 声明式隔离、Tasks 系统、CronCreate 等）见 **`docs/codex-adaptation.md`**。

## 维护约定（给课程开发者）

- **不要把教学协议复制到本文件**。如果某天你想"在 AGENTS.md 里改一下教学协议"，停——去改 `CLAUDE.md`。
- 本文件只在以下情况修改：(a) 新增 Claude/Codex 命令映射；(b) Codex 行为有专属差异需要点明；(c) `docs/codex-adaptation.md` 路径变了。
- 如果你发现 Codex 表现与 Claude 不一致，请先确认 Codex 是否真的读了 `CLAUDE.md`——大部分行为差异是"没读主指令"而不是"协议不同"。
