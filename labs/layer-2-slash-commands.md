# 层2 — 斜杠命令工程（Claude Code 的全部 `/xxx`）

> 可移植性：**中**（Codex 用 `$name` 触发 skill，没有自定义斜杠命令的对等机制；详见 `docs/codex-adaptation.md`）
> 路线提示：**A** 全做、慢节奏；**B** 跳过 §1（与层 1.5 重复），从 §2 开始；**C** §3 自定义 + §4 机制即可。

## 目标

把交互模式里的 `/xxx` 当**工程化对象**来掌握。学完你知道：哪些命令是 CLI 自带、哪些来自 skill/插件、哪些是你自己写的；写一个自定义命令需要什么文件结构、能传参吗、能跑 bash 吗、能跟权限/hooks 互动吗。

## 你将亲手产出

- `workspace/lab-02/slash-inventory.md` —— 本机斜杠命令清单（含来源：built-in / skill / plugin / custom）
- `.claude/commands/go.md` —— 你写的第一个自定义命令 `/go`：跑 git diff → 让 AI 总结 → 提示是否要 commit
- `.claude/commands/changelog.md` —— 第二个，带参数版：`/changelog v1.2..v1.3` 生成两个 tag 之间的更新说明

---

## 前置检查

- [ ] 完成层 1 / §1.5（知道斜杠命令只在交互模式生效，跑过 `/help` `/status` `/cost` `/context` 至少两个）
- [ ] 有一个能写 markdown 的编辑器（VS Code / Cursor / vim 都行）
- [ ] 工作区：`workspace/lab-02/`（教练帮你建）

---

## §1 内置斜杠命令分组详解（B 路线可跳）

> 进交互模式 `claude`，输入 `/help` 看本机实际列表。下面分组说明每组**该在什么场景反射性想到**。

### 1.1 会话生命周期
| 命令 | 干啥 | 反射性场景 |
|---|---|---|
| `/exit` `/quit` | 退出交互 | 聊完结束 |
| `/clear` | 清空当前 context | "换个话题" / context 太满变慢 |
| `/compact` | 压缩对话历史（保留摘要） | 接近 context 上限但不想丢 |
| `/resume` | 列出历史会话续接 | "接着上周那段聊" |
| `/context` | 当前 context 占用 | 觉得变慢、想知道还能塞多少 |

> **机制点**：`/clear` 物理清空，`/compact` 用 LLM 摘要。前者瞬时但丢细节，后者花一次模型调用换"记得大致经过"。

### 1.2 记忆与项目设置
| 命令 | 干啥 |
|---|---|
| `/memory` | 查看/编辑 `CLAUDE.md`（用户级 + 项目级） |
| `/init` | 让 Claude 扫一遍仓库生成项目 `CLAUDE.md` |
| `/add-dir` | 把额外目录纳入工作范围 |
| `/export` | 把当前会话导出 |

### 1.3 成本与状态
| 命令 | 干啥 |
|---|---|
| `/cost` | 当前会话累计 token / 美元 |
| `/usage` | 套餐用量与限额 |
| `/status` | 模型 / 目录 / 模式汇总 |
| `/doctor` | 自检（版本、网络、登录态、配置） |

### 1.4 配置与权限
| 命令 | 干啥 |
|---|---|
| `/config` | 设置面板 |
| `/model` | 切模型（Opus / Sonnet / Haiku） |
| `/permissions` | 工具权限（哪些命令可自动跑） |
| `/login` `/logout` | 切账号 |
| `/vim` | 切 vim 键位 |

### 1.5 高级能力入口（后续层会重点用）
| 命令 | 干啥 | 对应层 |
|---|---|---|
| `/agents` | 管理 subagent | 层 5 |
| `/hooks` | 管理 hooks（PreToolUse 等） | 层 3 |
| `/mcp` | 管理 MCP server | 层 4 |
| `/release-notes` | 看本版本更新 | 升级后看一眼 |
| `/bug` | 提 bug 给 Anthropic | 真遇到 bug |

### 动手 §1
进入 `claude` 跑 `/help`，把输出抄进 `workspace/lab-02/slash-inventory.md`，**用一列标注"我猜来源"**（built-in / skill / plugin / custom）。§4 我们会用 `/help` 的实际分组验证你猜得对不对。

---

## §2 Skill / Plugin 提供的斜杠命令

不是所有 `/xxx` 都是 CLI 自带——很多是**用 skill 或 plugin 注册的**。例如本机就装了一堆：

| 命令 | 来源 | 干啥 |
|---|---|---|
| `/loop` | skill (`loop`) | 把一个 prompt/skill 按间隔循环跑（`/loop 5m /code-review`） |
| `/verify` | skill | 启动 app + 在浏览器/CLI 验证改动确实工作 |
| `/code-review` | skill | 审 diff，分等级（low/medium/high/ultra） |
| `/simplify` | skill | `/code-review --fix` 的别名，直接改 |
| `/run` | skill | 启动项目 app（自动识别 CLI/server/TUI/web/Electron） |
| `/schedule` | skill | cron 风格的远程定时 agent |
| `/security-review` | skill | 安全视角的 diff 审查 |
| `/review` | skill | review 一个 PR（不止 diff） |
| `/claude-api` | skill | 协助构建/调试 Claude API 应用 |
| `/init` | skill | 初始化 `CLAUDE.md` |
| `/lab` `/labs` `/translate` | skill | **本课程**注册的 skill（你已用过） |
| `/update-config` | skill | 改 `settings.json` |
| `/keybindings-help` | skill | 键位配置 |
| `/fewer-permission-prompts` | skill | 减少权限弹窗（扫描历史生成 allowlist） |

> **机制点**：Skill 是带 frontmatter 的 markdown（`SKILL.md`），由 plugin 系统注入。CLI 启动时扫描 `~/.claude/plugins/marketplaces/*/plugins/*/skills/*` 和本仓库 `.claude/skills/`，每个 skill 注册一个同名斜杠命令。**用 skill 写一个斜杠命令 vs 用 `.claude/commands/` 写**——前者可被 LLM "主动调用"（看到关键词触发），后者只能用户**显式 `/`**。

### 动手 §2
1. 跑 `/loop --help`（或在 `/` 后输入 `loop` 看提示），看 skill 怎么自描述自己
2. 跑 `ls ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/ | head` —— 看你装了哪些 plugin（每个 plugin 可以塞多个 skill）

---

## §3 自定义斜杠命令（这一层的核心产出）

写一个文件就能造一个 `/xxx`。两个位置：

| 位置 | 范围 | 适用 |
|---|---|---|
| `<repo>/.claude/commands/foo.md` | 项目级（只在这仓库里能用） | 项目专属流程（`/lint` `/deploy-staging`） |
| `~/.claude/commands/foo.md` | 用户级（所有项目都能用） | 你的个人套路（`/standup` `/eod`） |

### 文件结构

```markdown
---
description: 一句话说清这个命令干啥（出现在 / 补全提示里）
argument-hint: <可选>给用户的参数提示，如 "[file-path]" 或 "[from-tag] [to-tag]"
allowed-tools: Bash, Read, Edit       # 可选：仅这些工具可被命令调用，缩小权限面
model: claude-haiku-4-5               # 可选：固定本命令用便宜模型省钱
---

这里是 prompt 正文，会作为用户消息发给 Claude。

支持几种插值：
- $ARGUMENTS — 整个参数串
- $1 $2 $3   — 位置参数
- !`shell命令` — 在 prompt 拼装前执行 shell，把输出嵌入
- @path/to/file — 引用文件内容（相对仓库根）
```

### 动手 §3.1 — 写你的第一个 `/go`

目标：`/go` = 看一眼工作区改动，让 AI 写一段中文总结 + 风险点。本质就是把层 1 的 `summarize.sh` 升级成斜杠命令。

```bash
mkdir -p .claude/commands
```

`.claude/commands/go.md`：

```markdown
---
description: 总结当前工作区改动并指出风险
allowed-tools: Bash
---

下面是当前的 git 改动：

!`git diff || git log -p -1 --no-color`

请用中文分两段总结：
1) 改了什么；
2) 潜在风险或需要复核的点。
末尾给一句"是否建议 commit / 还需要哪些手工验证"的判断。
```

跑：进入 `claude` → 输入 `/go` → 你应该看到 AI 直接拿着 diff 给出总结。

> **机制点**：
> - `!` 前缀让 CLI 在把 prompt 发给 Claude **之前**先跑 shell，把输出嵌进 prompt——这跟你在 shell 里 `git diff | claude -p "..."` 是等价的，但封装更整洁，而且可以纳入 `allowed-tools` 权限管控
> - `allowed-tools: Bash` = 即使这次会话默认要弹权限框，这个命令只会用到 Bash，不会乱碰文件

### 动手 §3.2 — 带参数的 `/changelog`

`.claude/commands/changelog.md`：

```markdown
---
description: 生成两个 git ref 之间的更新说明
argument-hint: <from-ref> <to-ref>
allowed-tools: Bash
---

下面是 $1 到 $2 之间的 commit log 和 diff stat：

!`git log --oneline $1..$2`

!`git diff --stat $1..$2`

请用中文写一份给用户看的"更新说明"，分三段：
- 新增功能
- 修复
- 不兼容变更（如有）

不要技术细节，写给非工程师能读懂的版本。
```

跑：`/changelog HEAD~3 HEAD`（或者两个真实 tag）。

> **机制点**：`$1 $2` 是位置参数；`argument-hint` 出现在 `/` 补全里给用户提示——这是斜杠命令版的"--help 文本"。

### 动手 §3.3 — 用户级命令（可选）

把 `go.md` 复制到 `~/.claude/commands/go.md`——这样你在**任何**项目里都能用 `/go`。如果同名命令同时存在用户级和项目级，**项目级优先**。

---

## §4 机制与坑

### 4.1 优先级与覆盖

`/help` 列出来的顺序通常是：内置 → 项目级自定义 → 用户级自定义 → skill/plugin。**同名时项目级压用户级，自定义压不掉内置**（你写 `/clear.md` 不会盖掉内置 `/clear`，但可能根本不被加载）。

### 4.2 跟 hooks 的关系

当你执行 `/go`，**会触发 `UserPromptSubmit` hook**——hook 看到的就是 prompt 拼好之后的完整文本（带 `!` 执行结果嵌入后的版本）。如果你想拦截 `/go` 做安全检查，目标就是这个 hook。层 3 会动手做。

### 4.3 跟权限的关系

- 命令 frontmatter 里的 `allowed-tools` 是**这个命令的白名单**——比当前会话的全局 permissions 更紧
- 没写 `allowed-tools` = 沿用会话默认权限
- 用户级命令在不同项目里跑，权限弹窗依然按**当前项目**的 permissions 设置走

### 4.4 何时该用 skill 而非 `.claude/commands/`

| 你要什么 | 用 |
|---|---|
| 用户主动 `/xxx` 触发的固定流程 | `.claude/commands/` |
| 希望 AI 在对话里"看到关键词就主动调用"的能力包 | skill |
| 含子目录、脚本、参考文档的复杂工具 | skill |
| 简单 prompt 模板 + 一两条 bash | `.claude/commands/` |

> 层 8 会教 skill 工程，对比就更具体。

### 4.5 调试技巧

- 命令"不出现在 `/` 补全里" → 检查文件名（不能带空格）和 frontmatter 是否合法 yaml
- 命令"跑了但什么都没发生" → frontmatter 的 `description` 缺失，或 prompt 正文是空的
- "为什么我的 `!` shell 命令没被执行" → 反引号 \`\` 必须紧贴 `!`，中间不能有空格；prompt 是 fenced code block 时 `!` 会被当字面量

---

## 完成标志

- ☑ 跑过 `/help`，能说出本机至少 5 个内置命令的用途
- ☑ 知道斜杠命令的三种来源（内置 / skill·plugin / 自定义）和它们的差别
- ☑ `.claude/commands/go.md` 和 `.claude/commands/changelog.md` 都能跑、产出合理
- ☑ 能讲清 `allowed-tools` / `argument-hint` / `!` shell 嵌入 / `$1` 参数 这 4 个机制点
- ☑ 能说出"`/go` 会触发哪个 hook"（→ 层 3 的伏笔）

学完更新 `workspace/progress.md`，然后 `/lab 3` 继续 harness 配置。

---

## 路线C 自测清单（10 分钟过关）

能独立完成以下全部，即可跳过本层进入层 3：

- [ ] 列出至少 3 个 skill 提供的斜杠命令（不能查 `/help`）
- [ ] 写一个 `.claude/commands/bisect.md`：参数是两个 commit hash，让 AI 帮你猜哪个引入了 bug
- [ ] 在命令里用 `allowed-tools` 把权限缩到只允许 `Bash(git:*)`（提示：specifier 语法见 settings 文档）
- [ ] 用 `/loop` 把 `/code-review` 每 30 分钟跑一次，并解释为什么这种用法消耗 token 但不一定有用
- [ ] 说清 skill 和 `.claude/commands/` 各自被 LLM 主动 vs 用户显式触发的差别
