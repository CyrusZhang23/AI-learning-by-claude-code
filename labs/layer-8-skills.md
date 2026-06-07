# 层8 — Skill 工程 + 虚拟项目实战（capstone）

> 可移植性：**高**（`SKILL.md` 格式 Claude/Codex 通用，触发语法 `/name` ↔ `$name`）

> 🎯 **本层一句话**：`description` 是给模型的**触发契约**（匹配素材），不是给人看的简介；确定性的活给脚本、需要判断的活给模型；分发是三层 —— skill（内容）→ plugin（包装）→ marketplace（货架）。

## 目标

这是整门课的毕业实战。在一个**虚拟项目**上，完整走一遍 skill 从设计到分发的生命周期：识别痛点 → 写 `SKILL.md` → 评估迭代 → 加内嵌工具 → 打包成 plugin → 安装触发。学完你能为任何重复性工作封装可复用的 skill。

## 你将亲手产出

- `workspace/lab-08/<虚拟项目>/` —— 一个有真实重复痛点的 toy 项目
- 一个自研 skill（`SKILL.md` + 内嵌零依赖工具脚本）
- 把它打包成 plugin，本地安装，用 `/<你的skill>` 触发验证

## 前置检查

- [ ] 安装了 `skill-creator` 插件（用于创建/评估/迭代 skill）
- [ ] 完成层2（理解自定义斜杠命令）和层5（理解 agent），层8 会综合用到
- [ ] 建好 `workspace/lab-08/` 工作区

## 分步教案

### 步骤 1 — 搭一个有痛点的虚拟项目
- **做什么**：建一个 toy 项目，要有**真实的重复性操作痛点**。推荐选一个：
  - **周报生成器**：每周要把一堆 git commit + 笔记整理成固定格式的周报
  - **读书笔记 CLI**：每读完一本书要按模板生成卡片、更新索引
  先把项目的"手动做一遍"流程跑通，亲身感受痛在哪。
- **为什么**：好 skill 来自真实痛点。先有痛，才知道 skill 要解决什么。
- **通过标准**：你能说清这个项目里"每次都要重复做、且有固定套路"的那个操作是什么。

### 步骤 2 — 用 skill-creator 设计 skill
- **做什么**：用 `skill-creator` 起一个 skill，针对步骤1 的痛点写 `SKILL.md`：name、description（**触发条件要写准**——这决定 skill 何时被自动调用）、正文步骤。
- **为什么**：`SKILL.md` 是 skill 的契约。description 写不好，skill 该触发时不触发、不该触发时乱触发。
- **通过标准**：你有一份结构完整的 `SKILL.md`，description 能准确描述"什么时候该用我"。

### 步骤 3 — 加内嵌工具脚本（零依赖原则）
- **做什么**：给 skill 配一个内嵌脚本（如 `generate.py`，把输入按模板渲染成成品），遵循**零依赖/最小依赖**原则，让 skill 拷过去就能用。
- **为什么**：顶级 skill 项目的共识——内嵌工具脚本降低安装摩擦，零依赖才好分发。
- **通过标准**：skill 调用内嵌脚本能产出成品，且脚本不依赖一堆要额外装的包。

### 步骤 4 — 评估与迭代
- **做什么**：用 `skill-creator` 的评估能力（或自己设计几个测试场景）检验 skill：触发准不准？产出对不对？边界情况处理了吗？根据结果改 `SKILL.md` 和脚本，迭代 1–2 轮。
- **为什么**：skill 工程的核心是迭代。第一版几乎不可能完美。
- **通过标准**：迭代后，skill 在你的测试场景里触发正确、产出可用。

### 步骤 5 — 打包成 plugin 并安装
- **做什么**：把 skill 按 plugin 结构打包（`.claude-plugin/plugin.json` → `marketplace.json`），`claude plugin marketplace add <本地路径>` + `claude plugin install <名>@<market>`，在**仓库外的干净目录**重启会话验证。
- **▶ 让学员先预测（description 契约的终极验收）**：在干净环境里，问学员——"我**不打 `/你的skill`**，而是用自然语言说'帮我把这些碎片聚合成发版说明'，skill 会被唤起吗？" **揭晓**：会——只要 `description` 写准，模型看到关键词就主动调用。这就是步骤 2 那句"description 是触发契约不是简介"的终极证明（对比 `.claude/commands/` 只能显式 `/`，呼应层 2）。再问一题考 token 经济学："plugin 装上后，`description` 和正文哪个一直占着 context、哪个按需才加载？"（答案：description 是 always-on 常驻税~160 token，正文 on-invoke 按需~730——所以 description 要短而准。）
- **为什么**：plugin 是 skill 的分发单元；自然语言触发成功 = description 契约写对了。
- **通过标准**：在本仓库之外的目录里，**不打斜杠命令、用自然语言**也能唤起 skill 并正常工作。

### 步骤 6 — 对照顶级项目迭代质量
- **做什么**：对照 `docs/skill-projects.md` 里的设计洞察（如官方 skills 的结构规范、superpowers 的方法论、零依赖原则），回看自己的 skill 还能怎么改进。
- **为什么**：站在巨人肩上——成熟项目的设计模式能直接提升你的 skill 质量。
- **通过标准**：你能列出至少 2 个从顶级项目学来、可应用到自己 skill 的改进点。

### 步骤 7 — 全景扫描（收尾 + 毕业回望）[核心]
- **做什么**：① skill 维度——对照分发三层（skill / plugin / marketplace）和 `claude plugin` 子命令全集（`marketplace add/remove`、`install`、`uninstall`），确认每层职责；② **全课维度**——带学员把 8 层的"一句话原理锚点"串一遍（CLI 两入口 → 斜杠三来源 → harness 三件套 → MCP 协议分工 → 子 agent 独立闸门 → 文件总线+独立裁判 → 三层洋葱 → description 契约），看它们怎么环环相扣。
- **为什么**：这是毕业层，全景扫描不只查本层漏，更是把整门课的主线收成一张图。

> **🧪 留个回归夹具（superpowers 启发）**：把步骤 4 评估时用的临时测试场景**固化成 `tests/` 夹具**——以后改 skill / 改 `SKILL.md` 后，跑一遍夹具就能回归"触发准不准、产出对不对"。这是把"一次性测试"升级成"可重复回归闸门"，正是顶级 skill 项目的做法。

> **⚠️ 清理提示**：本层为演示装过 user-scope plugin（如 `changelog-tools@lab08-local`）和 `~/tmp` 测试目录。学完可还原：`claude plugin uninstall <名>` + `claude plugin marketplace remove <market>`，删掉临时测试目录。

## 完成标志

- 你独立完成了一个虚拟项目 + 一个解决其痛点的自研 skill
- skill 带内嵌工具、能被打包成 plugin、能在干净环境**用自然语言**触发
- 你能讲清"一个好 skill 的 description 为什么是成败关键"
- 🎯 能复述本层那句话：description 是触发契约不是简介；确定性给脚本、判断给模型；分发三层
- **毕业**：你已走完从 CLI 到 Skill 工程的全部 9 层（0–8，含摸底层 0），能复述每层的一句话主线

## 延伸（可选）

- 把你的 skill 发布到一个公开仓库，让别人能安装
- 设计一个"skill 编排 skill"的工作流（一个 skill 调用另一个），呼应 superpowers 的方法论
