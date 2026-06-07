# 层5 — 多 Agent 协同（单实例内）

> 可移植性：**中**（worktree 声明式隔离、Tasks 系统 Codex 无直接对应，见 `docs/codex-adaptation.md`）

> 🎯 **本层一句话**：子 agent = 独立上下文 + 权限边界（`tools:` 字段）；而**自检闸门必须独立于干活方** —— 让写代码的 agent 自己说"我测过了没问题"等于没审，必须另起一个只读 agent 来卡。

## 目标

在**一个 Claude Code 实例内部**编排多个 agent：派发子 agent、并行执行、用 worktree 隔离工作区、用 Tasks 跟踪长任务、用自主循环让 agent 持续推进。这是"多 AI 协作"的内功。

## 你将亲手产出

- 一份编排实验记录 `workspace/lab-05/orchestration.md`
- 一个由 worktree agent 在独立分支产出的改动
- 一段用 Tasks 跟踪的多步任务执行轨迹

## 前置检查

- [ ] 当前目录是一个 git 仓库（worktree 实验需要）；若不是，先 `git init`
- [ ] 理解层1 的子 agent 概念
- [ ] 建好 `workspace/lab-05/` 工作区，里面放几个示例代码文件供 agent 探索

## 分步教案

### 步骤 1 — 派发只读探索 agent
- **做什么**：用 Agent 工具派一个 `Explore` 类型子 agent，让它"找出 workspace/lab-05 下所有 Python 文件里定义的函数并汇总"。观察它返回的是**报告**，不是直接改你的上下文。
- **为什么**：理解 subagent 的隔离性——它有独立上下文，只把结论带回来，保护主上下文不被搜索噪音污染。
- **通过标准**：你拿到一份探索报告，且能说出"为什么用 Explore 而不是自己 grep"。

### 步骤 2 — 理解 subagent 类型
- **做什么**：对比 `Explore`（只读搜索）、`Plan`（只读分析）、`general-purpose`（可写多步）三种类型，各派一个小任务感受差异。
- **为什么**：选对 agent 类型 = 给它恰当的权限和职责边界。
- **通过标准**：你能说出三种类型各自适合什么任务。

### 步骤 3 — 定义你自己的子 agent 工种（自定义 agent）

> **先分清两种"subagent"，别混：**
>
> | | A. Agent 工具（临时工） | B. 自定义 agent 定义（正式工种） |
> |---|---|---|
> | 存在形式 | 运行时临时起，干完消失 | `.claude/agents/<名字>.md` 文件，预先存在 |
> | 角色/提示词 | 调用时现写 prompt | **预先写在 .md 正文里** |
> | 权限边界 | 选内置类型（Explore/Plan/...） | frontmatter 的 `tools:` 字段 |
> | 比喻 | 招个临时工干一票 | 设一个正式编制岗位 |
>
> 步骤 1–2 用的是 A（内置工种）。这步做 B：**预先定义一个专长固定的子 agent**，以后按名字反复调用。

- **做什么**：在 `.claude/agents/` 下写自定义 agent 定义文件（每个一个 `.md`，带 frontmatter）。例如两个专长 agent：`todo-hunter`（专找 TODO）、`docstring-auditor`（专查缺 docstring 的函数）。文件结构：
  ```markdown
  ---
  name: todo-hunter
  description: 专门扫描代码里的 TODO/FIXME 并汇总（看到"找TODO"类请求可主动触发）
  tools: Read, Grep, Glob        # 只读工具 = 权限边界，它改不了文件
  model: sonnet
  ---
  你是 TODO 猎手。任务：在给定目录里找出所有 TODO/FIXME 注释，
  逐条报告：文件、所在函数、原文。只读，绝不修改任何文件。
  ```
- **为什么**：自定义 agent = 把"角色 + 权限 + 提示词"固化成可复用工种。`tools:` 字段就是它的权限边界（跟层3 的 permissions、层4 的 MCP 工具是同一套"最小权限"思想）。专长写在提示词正文里。
- **通过标准**：`.claude/agents/` 下有你的 agent 定义，`/agents` 能看到它，且主 agent 能用 `subagent_type=<你的agent名>` 调用它。

> **A 与 B 怎么连起来**：主 agent 正是通过 A（Agent 工具）去调用 B（你定义的工种）的——`subagent_type` 参数除了能填内置的 `Explore`，**也能填你自定义的 agent 名字**。B 是"定义工种"，A 是"派活的动作"。

> **给自定义 agent 配 skill（官方文档核实）**：不是靠"私有 skill 文件夹"，是靠 frontmatter 的 **`skills:` 字段**按名字引用：
> ```markdown
> ---
> name: todo-hunter
> tools: Read, Grep, Glob
> skills: [find-todos]      # ← 引用 skill 名；skill 本体仍住在 .claude/skills/ 或插件
> ---
> ```
> **skill 是共享的，不是某个 agent 私有的**。完整 skill 工程见层8。

> **关于"文件夹"——一个常见误解，务必厘清（官方文档核实）**：
>
> - **agent 永远是一个 `.md` 文件，不是文件夹。** 文件夹里必须有 `.md`，空文件夹什么都不是。
> - 你**可以**在 `.claude/agents/` 下建子文件夹（如 `agents/research/foo.md`）——但那**纯粹是整理归类用**。agent 的身份只看 `.md` 里 frontmatter 的 `name`，**和它在哪个文件夹无关**。
> - 所以**"一个文件夹 = 一个带自己 CLAUDE.md 和私有 skill 的 agent"，原生机制里不存在**。
> - 例外：**插件**的 `agents/` 子文件夹会进入标识符（`agents/review/security.md` 在插件 `my-plugin` 里注册成 `my-plugin:review:security`）——但仍是"`.md` 文件是 agent"，只是多了命名空间前缀。
>
> | 形态 | 位置 | 是不是"文件夹=agent" |
> |---|---|---|
> | ① 单文件·项目级 | `<repo>/.claude/agents/foo.md` | 否，agent 是 .md |
> | ② 单文件·用户级 | `~/.claude/agents/foo.md` | 否，agent 是 .md |
> | ③ 子文件夹整理 | `.claude/agents/research/foo.md` | 否，文件夹仅归类，身份看 name |
> | ④ 打包进插件 | `<plugin>/agents/foo.md` | 否，但插件子文件夹会进标识符 |
>
> **审查者要点**：看到任何"自定义 agent"，先看那个 `.md` 的 frontmatter——`tools:`（权限边界）+ `skills:`（扩展能力）——这决定它能干什么、危不危险。

### 步骤 4 — 并行派发
- **做什么**：在一条消息里同时派两个 agent（如一个查"有哪些 TODO 注释"，一个查"有哪些函数没有 docstring"），让它们并行跑。
- **为什么**：独立任务并行能显著提速，且各自上下文隔离。
- **通过标准**：两个 agent 并行返回，你拿到两份独立结果。

### 步骤 5 — Worktree 隔离
- **做什么**：用 `isolation: "worktree"` 派一个 agent，让它在独立 git 分支/工作树里改一个**已被 git 跟踪的文件**。完成后查看它产生的分支和改动，主工作区不受影响。
- **为什么**：worktree 隔离让 agent 的实验性改动不污染你当前工作区——失败可弃、成功可合。
- **通过标准**：改动发生在独立分支，你的主工作区 `git status` 干净。

> **机制**：`git worktree` 是纯 git 功能——从仓库 checkout 出第二个独立工作目录、在新分支上，与主区互不干扰。`isolation: "worktree"` 让 agent 自动开「新分支 + 新工作树」，在里面改，干完报告分支名，你再决定合不合并。这跟层4 的「权限最小化」同源：把风险**隔离**起来。

> **⚠️ 真实坑（▶ 先让学员预测）：worktree 只能隔离 git _跟踪_ 的文件**。
>
> 派 worktree agent 前先问学员——"如果我让这个隔离 agent 去改 `workspace/` 下一个文件（`workspace/` 是 gitignored 的），它会改在独立工作树里、不碰我主区吗？" 多数人答"会，这就是隔离的意义"。**揭晓：不会**，改动会落到你主工作区，隔离是假象。原因往下看：
> `git worktree` 的新工作树**只包含已提交(tracked)的文件**；gitignored / 未跟踪的内容根本不进工作树。所以如果你让 agent 改一个 **gitignored 文件**（比如本课程 `workspace/` 就是 gitignore 的学习沙盒）：
> - 隔离工作树里**没有那个文件**（它不在任何 commit 里）
> - agent 找不到，就改到了**唯一存在的那份——主工作区的**
> - 改完隔离树「tracked 文件无变化」，harness 把空工作树自动清理 → **看起来用了隔离，实际改动落在主区，隔离是假象**
>
> | 想隔离的内容 | worktree 有效吗 |
> |---|---|
> | 已提交的源码(tracked) | ✅ 有效，给独立分支 |
> | gitignored / 未跟踪文件 | ❌ 无效，改动落主区 |
>
> **审查者判断**：用 worktree 隔离前先问一句——「这个文件 git 跟踪了吗？」没跟踪，隔离就是假的。要演示真隔离，必须挑一个 **tracked 文件**。

### 步骤 6 — Tasks 系统
- **做什么**：用 TaskCreate 把一个三步任务（探索 → 修改 → 验证）建成任务列表，逐步 `in_progress` → `completed`，用 TaskList 查看进度。
- **为什么**：Tasks 是长任务的状态追踪机制，让多步/长流程可见、可恢复。
- **通过标准**：你能看到任务从 pending 走到 completed 的完整轨迹。

### 步骤 7 — 自主循环
- **做什么**：用 `/loop` 或 `ralph-loop` 让 agent 自主迭代一个有明确完成条件的任务（如"逐个给 workspace/lab-05 的函数补 docstring，直到全部补完"）。观察它如何自我推进、何时停止。
- **为什么**：理解自主循环的"目标 → 迭代 → 自检 → 停止"模式，以及它和一次性任务的区别。
- **通过标准**：循环在达成完成条件后正确停止，产出符合预期。

### 步骤 8 — 全景扫描（收尾）[核心]
- **做什么**：对照本层的能力面问学员——① subagent 类型全集（`Explore` / `Plan` / `general-purpose` / `claude-code-guide` / 自定义）各自定位；② agent frontmatter 字段全集里我们只用了 `tools` / `skills`，还有 `model` / `permissionMode` / `isolation` / `background` 等没碰；③ Tasks 系统除 Create/Update/List 还有 Get/Output/Stop。明确"这层学了哪些、留了哪些"。
- **为什么**：agent 的可配置面很大，建立全景索引避免以为"会派 agent"就是全部。

> **🧪 留个回归夹具**：把步骤 7 的两段式流水线（docstring-auditor 只读自检闸门 → general-purpose writer 补 → 复审清零停止）连同 `lab-05/text_utils.py` 留作夹具——它同时验证"自检闸门独立于干活方"这条铁律。改教案后重跑一遍，确认循环仍能在达标时正确停止。

## 完成标志

- 你能派发单个、并行、隔离（worktree）三种形态的 agent
- 你用 Tasks 完整跟踪过一个多步任务
- 你跑通了一个会自动停止的自主循环
- 你能讲清"主 agent vs 子 agent 的上下文隔离"带来的好处
- 🎯 能复述本层那句话：子 agent = 独立上下文 + 权限边界；自检闸门必须独立于干活方

## 延伸（可选）

- 用 CronCreate 建一个定时 agent（如每天总结改动）——这会接到层6
- 设计一个"探索 agent 产出报告 → 实现 agent 据报告改代码"的两段式流水线
