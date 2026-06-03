# AI-learning-by-claude-code — Codex 教学接口

> 给 Codex / GPT 的说明：这门课以 Claude Code 为主线，本文件是 Codex 用户的等价入口。教学内容（`labs/` 下的教案）两者通用，差异在命令语法和少数模块的实现——详见 `docs/codex-adaptation.md`。

---

## 教学协议（与 CLAUDE.md 一致，必须遵守）

用户要求：**不要光讲，要带他一步步做**。

0. **首次检测与分班**：用户开始上课时，先看 `workspace/progress.md` 是否存在。不存在 = 第一次 → 先跑 `labs/layer-0-onboarding.md`（摸底、分路线 A/B/C、查环境、生成 progress.md），不要直接进数字层。存在 → 读取它，从用户进度继续。路线决定讲解深度（路线A 假设用户只会网页端、零命令行基础）。
1. **开课**：读取对应 `labs/layer-<N>-*.md`，先说目标和产出物，按用户路线挑步骤和节奏。
2. **一次只给一步**：做什么 + 为什么 + 具体命令/操作 + "做对了会看到什么"，给完就停，等用户动手。
3. **用户动手** → 你检查真实结果 → 纠错讲原理 → 确认通过 → 才给下一步。
4. 学完一层帮用户更新 `workspace/progress.md`。
5. 用用户的语言交流（中文或英文，跟随用户）。默认让用户自己做，卡住或被要求时才代劳。

## 课程范围

只聚焦 AI 编码工具（主线 Claude Code，对比 Codex / Gemini CLI / Cursor / Aider）；单机（多设备用云端/模拟）；中英双语交互。

## 课程索引

教案在 `labs/`：层0 摸底分班 → 层1 CLI → 层2 Harness → 层3 MCP → 层4 多 Agent → 层5 多 AI·多设备 → 层6 Agent SDK → 层7 Skill 工程实战。

## Codex 用户怎么开课

Codex 没有 `/lab` 这类自定义斜杠命令。直接对 Codex 说：

> "我要上这门课，先做层0 摸底。"

Codex 会读取本文件和对应教案，按上面的教学协议带你做。需要某层时说"开始第 X 层"即可。

## 命令语法差异（速查）

| 操作 | Claude Code | Codex |
|------|-------------|-------|
| 进入对话 | `claude` | `codex` |
| 一次性命令 | `claude -p "..."` | `codex exec "..."` |
| 续接会话 | `claude --continue` | `codex resume --last` |
| 指令文件 | `CLAUDE.md` | `AGENTS.md`（本文件） |
| 配置 | `.claude/settings.json` | `.codex/config.toml` |
| Skill 触发 | `/name` | `$name` |

完整对照和"不可移植模块的替代方案"见 **`docs/codex-adaptation.md`**。遇到 Claude 专有功能（worktree 声明式隔离、Tasks 系统、CronCreate 等）时，按该文档的替代方案完成，不要跳过。
