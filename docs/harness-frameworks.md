# 开源 Harness 框架对照（层2 延伸阅读）

学习 Claude Code 的 harness（settings / hooks / 权限 / 生命周期）时，横向对比这些开源 Agent 框架能加深理解——它们各自用不同方式解决了同一类"如何安全地拦截和编排 AI 工具调用"的问题。

| 框架 | 与 Claude Code harness 的对应关系 |
|------|----------------------------------|
| **LangGraph** (langchain-ai) | 最接近：checkpointed 可恢复状态 + interrupt/resume ≈ 会话续接；graph lifecycle hooks ≈ PreToolUse/PostToolUse |
| **CrewAI** (crewAIInc) | Flows 拦截点 ≈ hooks；YAML crew 配置 ≈ settings.json；MCP 作为一等公民工具源 |
| **AutoGen / ag2** (ag2ai) | HumanProxyAgent 审批门 ≈ 权限弹窗；工具显式注册 + 权限作用域 ≈ settings allowlist |
| **LlamaIndex Workflows** (run-llama) | PreToolUse hook 语义最直接对应同名 hook；事件驱动步骤 ≈ harness 生命周期 |
| **Semantic Kernel** (microsoft) | Kernel + Plugin 自动发现 ≈ harness + SKILL.md；Function Filters ≈ Pre/PostToolUse hooks |
| **OpenHands** (OpenHands) | Action-Observation 循环 + policy dispatcher ≈ harness 工具拦截；最严格的沙箱权限模型 |
| **Agency Swarm** (VRSEN) | Agent manifest（角色 + 允许工具）≈ CLAUDE.md + settings.json；轻量级入门首选 |
| **Haystack** (deepset-ai) | YAML pipeline 配置 ≈ settings.json 工具连接；typed I/O 合约 ≈ 工具 schema 校验 |

**怎么用这张表**：不必全学。学完层2 后，挑 1 个框架（推荐先看 LangGraph 的 checkpoint/interrupt，或 OpenHands 的沙箱权限模型），读它对应特性的文档，对照你刚在 Claude Code 里配的 hooks/权限，理解"同一个 harness 问题的不同解法"。
