# 层2 — 斜杠命令工程（Claude Code 的全部 `/xxx`）

> 可移植性：**中**（Codex 用 `$name` 触发 skill，没有自定义斜杠命令的对等机制；详见 `docs/codex-adaptation.md`）
> 路线提示：**A** 全做、慢节奏；**B** 跳过 §1（与层 1.5 重复），从 §2 开始；**C** §3 自定义 + §4 机制即可。

> 🎯 **本层一句话**：斜杠命令有三种来源（内置 / skill·plugin / 自定义）；自定义命令就是一个 `.md` 文件 —— 关键分水岭是 skill 能被 LLM"看到关键词主动调用"，`.claude/commands/` 只能用户**显式 `/` 触发**。

## 目标

把交互模式里的 `/xxx` 当**工程化对象**来掌握。学完你知道：哪些命令是 CLI 自带、哪些来自 skill/插件、哪些是你自己写的；写一个自定义命令需要什么文件结构、能传参吗、能跑 bash 吗、能跟权限/hooks 互动吗。

## 你将亲手产出

- `workspace/lab-02/slash-inventory.md` —— 本机斜杠命令清单（含来源：built-in / skill / plugin / custom）
- `.claude/commands/go.md` —— 你写的第一个自定义命令 `/go`：注入环境快照（时间/目录/Python 版本），让 AI 写一句今日开工摘要
- `.claude/commands/explain.md` —— 第二个，带参数版：`/explain <术语>` 用一段话解释一个技术术语

---

## 前置检查

- [ ] 完成层 1（知道交互模式 vs headless 的区别）
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
| `/btw` | 问个不污染主线的旁问 | 主任务跑一半临时想确认 API 用法、看个文档片段、查个命令 |
| `/rewind` | 把代码/对话回滚到某个点 | 改坏了想"撤回"到上一个 checkpoint |
| `/branch` | 在当前对话点开"分支" | 想试两种不同方案对比,不想把主线搅乱 |
| `/diff` | 看未提交改动 + 每轮 diff | 跟 `/go` 是天作之合,review 自己改动不切终端 |
| `/recap` | 给当前 session 一句话总结 | 想 `/clear` 之前抓个快照 |
| `/rename` | 重命名当前会话 | `/resume` 找历史时一眼能认出 |
| `/copy` `/copy N` | 抄 AI 最近第 N 条回答到剪贴板 | 日常高频,免得拖选乱码 |

> **机制点 1**:`/clear` 物理清空,`/compact` 用 LLM 摘要。前者瞬时但丢细节,后者花一次模型调用换"记得大致经过"。
>
> **机制点 2(context 隔离三件套)**:`/btw` `/branch` `/rewind` 都是在 **不破坏主线 context** 的前提下做事——
> - `/btw` = 临时开一个旁支对话问一句,回来后**旁支问答不进主对话历史**,主任务的 context 没被稀释
> - `/branch` = 从当前点 fork 一条平行对话,两条独立演进,可来回切
> - `/rewind` = 时间机器,把对话(可选连同代码 checkpoint)倒回某个点重来
>
> 三个加起来是 Claude Code 区别于"线性聊天框"AI 的核心:**对话状态可分叉、可回滚、可侧问**。后面层 5 多 agent 协同会再用到这个思路。

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
| `/fast` | 切 fast mode（Opus 4.7,输出更快）——你已经在用 |
| `/effort` | 调当前模型的 effort level（low/med/high/xhigh）——同模型也能"档位切换",成本/质量旋钮 |
| `/focus` | 只显示你 prompt + 工具摘要 + 最终回复——屏幕清爽,跟 verbose 反着的 |

### 1.5 高级能力入口（后续层会重点用）
| 命令 | 干啥 | 对应层 |
|---|---|---|
| `/agents` | 管理 subagent | 层 5 |
| `/hooks` | 管理 hooks（PreToolUse 等） | 层 3 |
| `/mcp` | 管理 MCP server | 层 4 |
| `/release-notes` | 看本版本更新 | 升级后看一眼 |
| `/bug` | 提 bug 给 Anthropic | 真遇到 bug |
| `/plan` | 开 plan 模式 / 看当前会话计划 | 设计型任务必用,先规划再执行 |
| `/goal` | 设目标,让 AI 一直干到达成 | 自主循环的**内置**入口,层 5 自主 agent 会重用 |
| `/advisor` | 关键节点叫更强模型给意见 | Opus 主跑 + 关键时刻问 ultra 的混合模式,省钱+提质 |
| `/background` `/tasks` | 把 session 丢后台 / 看后台任务列表 | 长跑任务释放终端,层 5 重度用 |

### 动手 §1（5 分钟，先建立直觉）

进入 `claude` 交互模式后依次跑这 4 条，注意你看到了什么：

1. **`/help`** —— 列你这一版实际支持的全部命令
2. **`/status`** —— 当前会话状态（模型、目录、permission 模式）
3. **`/cost`** —— 这次会话累计花了多少
4. **`/context`** —— context 占用条

然后 **▶ 让学员先预测来源**：教练把 `/help` 的完整列表拉成 `workspace/lab-02/slash-inventory.md`（教练生成，不让学员逐条手抄），但**留一列空着让学员口头猜每个命令的来源**（built-in / skill / plugin / custom）。学员猜、教练记，§4 用实际机制验证猜得对不对。重点不是"抄全这张表"（那是低价值机械活），而是"你能不能凭直觉判断一个命令是 CLI 自带还是 skill 注册的"。

> 关键事实（一行就够）：斜杠命令**只在交互模式生效**；在 shell 里直接打 `/cost` 没用。

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
| `/batch` | skill | 规划一个大改动,**并行 5–30 个实例**执行——层 5 多 agent 的实战形态 |
| `/autofix-pr` | skill | 监控当前 PR + 自动修问题 |
| `/debug` | skill | 当前会话开 debug log,排查 hook/MCP 卡壳必用 |
| `/insights` | skill | 出一份分析你 Claude Code 历史的报告 |
| `/statusline` | skill | 配状态栏 UI,跟层 3 hooks 互补 |
| `/team-onboarding` | skill | 用你的使用习惯给队友生成上手指南 |

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

目标：`/go` = 把当前环境信息（时间 + 目录 + Python 版本）注入 prompt，让 AI 写一句"今日开工摘要"。选这个例子是因为 `date`/`pwd`/`python3 --version` 在任何机器上都能跑，不依赖 repo 状态。

```bash
mkdir -p .claude/commands
```

`.claude/commands/go.md`：

```markdown
---
description: 注入环境快照，让 AI 写今日开工摘要
allowed-tools: Bash
---

当前环境快照：
- 时间：!`date "+%Y-%m-%d %H:%M"`
- 目录：!`pwd`
- Python：!`python3 --version 2>&1`

根据上面的信息，用一句话写一个"开工提示"，风格轻松，带当前时间。
```

**▶ 让学员先预测**：跑 `/go` 之前先问——"这三行 `!\`date\`` / `!\`pwd\`` 是谁来执行？是发给 Claude 让它去跑，还是 CLI 在发出去之前自己先跑掉、把结果替换进文本？" 让学员答完再跑，对照验证。（答案：CLI 先跑，Claude 收到的已经是替换好结果的纯文本。）

跑：进入 `claude` → 输入 `/go` → 你应该看到 AI 拿着三行环境信息生成一句开工摘要。

> **机制点**：
> - `!` 前缀让 CLI 在把 prompt 发给 Claude **之前**先跑 shell，把输出嵌进 prompt
> - 多个 `!` 行按顺序执行，各自独立，输出原位替换
> - `allowed-tools: Bash` = 这个命令只会用到 Bash，不会乱碰文件

### 动手 §3.2 — 带参数的 `/explain`

目标：`/explain <关键词>` = 用户传一个词，AI 用一段话解释它是什么。重点是学 `$ARGUMENTS` 和 `argument-hint`，不依赖任何外部状态。

`.claude/commands/explain.md`：

```markdown
---
description: 用一段话解释一个技术术语
argument-hint: <术语>
---

请用中文、面向有一点编程基础的读者，用不超过 3 句话解释：$ARGUMENTS

要求：第一句说"是什么"，第二句说"用在哪"，第三句举一个最小例子。
```

跑：`/explain PreToolUse hook`

> **机制点**：`$ARGUMENTS` 是整个参数串（不拆分）；`argument-hint` 只影响 `/` 补全时显示给用户的提示文字，不参与执行。`/explain` 没有 `!` 行，所以 CLI 不执行任何 shell——prompt 直接发给 Claude，`$ARGUMENTS` 替换后就是用户输入的词。

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

## 收尾 — 全景扫描（核对预测 + 查漏）[核心]

回到 §1 的 `slash-inventory.md`：教练带学员把当初"猜来源"那一列和 §4 学到的实际机制**逐条对一遍**——猜对的过、猜错的讲为什么。再扫一眼 `/help` 里**整层没动过的命令分组**（如 `/agents`→层5、`/hooks`→层3、`/mcp`→层4），明确"这些存在、留到后面层"。这满足"我知道还有什么没学"的全景感。

## 完成标志

- ☑ 跑过 `/help`，能说出本机至少 5 个内置命令的用途
- ☑ 知道斜杠命令的三种来源（内置 / skill·plugin / 自定义）和它们的差别
- ☑ `.claude/commands/go.md` 和 `.claude/commands/explain.md` 都能跑、产出合理
- ☑ 能讲清 `allowed-tools` / `argument-hint` / `!` shell 嵌入 / `$1` 参数 这 4 个机制点
- ☑ 能说出"`/go` 会触发哪个 hook"（→ 层 3 的伏笔）
- ☑ 🎯 能复述本层那句话：三种来源 + "skill 可被 LLM 主动调用 vs commands 只能显式 /"

学完更新 `workspace/progress.md`，然后 `/lab 3` 继续 harness 配置。

---

## 路线C 自测清单（10 分钟过关）

能独立完成以下全部，即可跳过本层进入层 3：

- [ ] 列出至少 3 个 skill 提供的斜杠命令（不能查 `/help`）
- [ ] 写一个 `.claude/commands/bisect.md`：参数是两个 commit hash，让 AI 帮你猜哪个引入了 bug
- [ ] 在命令里用 `allowed-tools` 把权限缩到只允许 `Bash(git:*)`（提示：specifier 语法见 settings 文档）
- [ ] 用 `/loop` 把 `/code-review` 每 30 分钟跑一次，并解释为什么这种用法消耗 token 但不一定有用
- [ ] 说清 skill 和 `.claude/commands/` 各自被 LLM 主动 vs 用户显式触发的差别
