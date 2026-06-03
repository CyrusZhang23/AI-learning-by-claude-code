# 10 个热门开源 Skill 项目设计分析（层7 延伸阅读）

学层7 Skill 工程时，这些社区项目是最好的学习素材——看它们怎么组织 skill、怎么设计触发、怎么分发。

| # | 项目 | 设计亮点 | 对自建 skill 的启发 |
|---|------|---------|-------------------|
| 1 | **everything-claude-code** (affaan-m) | 大量 agents + skills + hook profiles；编排引擎；环境变量门控 hook profile | hook profile 分层：开发/生产/审计模式可切换 |
| 2 | **anthropics/skills**（官方） | 四分类体系（创意/开发/企业/文档）；docx/pdf/pptx/xlsx 文档 skill 最实用 | 官方规范：`SKILL.md` 结构的黄金标准 |
| 3 | **awesome-claude-code** (hesreallyhim) | 生态索引：skills/agents/hooks/MCP 全覆盖；人工品控 | 发现新 skill 的首选入口 |
| 4 | **antigravity-awesome-skills** (sickn33) | 上千 skills；机器可读的 `skills_index.json`；跨平台 | 索引设计：大规模 skill 库需要结构化元数据 |
| 5 | **superpowers** (obra) | 方法论驱动：`brainstorming→writing-plans→executing-plans→TDD` 四阶段自动触发；subagent-driven-development | **skill 可以编排其他 skill，形成工作流链** |
| 6 | **alirezarezvani/claude-skills** | 多领域 skills + 大量零依赖 Python 工具 + 可安装插件 | 零依赖原则：skill 内嵌工具脚本，降低安装摩擦 |
| 7 | **anthropics/knowledge-work-plugins**（官方） | 按职业角色打包（销售/产品/工程/设计…）；对接真实 SaaS（Notion/Linear/Figma） | 按角色打包而非按功能：企业场景的分发单元 |
| 8 | **claude-code-plugins-plus-skills** (jeremylongshore) | 大量 plugins + skills；配套 CLI 包管理器 | 包管理器是大规模 skill 生态的基础设施 |
| 9 | **claude-forge** (sangrokjung) | oh-my-zsh 风格；多层安全 hook 系统 | 安全 hook 分层：比单点拦截更健壮的防御纵深 |
| 10 | **awesome-claude-code-toolkit** (rohitg00) | 一批 hook 脚本（破坏性命令拦截/权限自动批准/上下文保存） | hook 脚本集合：最好的 hook 学习素材库 |

## 三个关键洞察

1. **顶级项目都有方法论，不只是 prompt 集合**——superpowers 的四阶段流程是典型。skill 的价值在于固化"做事的套路"，不只是固化"一句提示"。
2. **跨平台兼容（Cursor/Codex/Gemini）是大型 skill 库的标准配置**——`SKILL.md` 格式的通用性是基础。
3. **零依赖 / 最小依赖是 skill 可分发的前提**——内嵌工具脚本、不要求用户额外安装，才能"拷过去就能用"。

> 做层7 实战时，做完自己的 skill 后回看这张表，挑 2 个洞察应用到你的 skill 上（步骤6）。
