# AI-learning-by-claude-code

**简体中文** | [English](./README.en.md)

> 一套**动手式**的 AI 编码工具系统课程。克隆下来，在仓库里打开 Claude Code，输入 `/lab`，就有一位 AI 教练**带你一步步做**——第一次上课会先摸底分班，之后每个功能都让你亲手跑通，而不是看文档。

聚焦 **AI 编码工具**（以 Claude Code 为主线，横向对比 Codex / Gemini CLI / Cursor / Aider）。课程含一节开课摸底（层0）+ 7 层由浅入深的实战：CLI、Harness 配置、MCP、多 Agent 协同、多 AI 协作、Agent SDK 和 Skill 工程。

## 这门课的特别之处

- **零基础友好**：默认你只用过网页版 AI、没碰过命令行。第一次上课（层0）会问几个问题摸底，把你分到合适的路线（零基础 / 有基础 / 进阶），后面按你的水平来。
- **教练式，不是讲座**：每一步你自己动手（敲命令、写文件），AI 检查你的结果、解释原理，确认掌握了才进入下一步。
- **clone 即用**：仓库自带教学接口（`CLAUDE.md` + `/lab` 命令），无需任何额外配置。
- **每层都有产出物**：学完不是"看过了"，而是手里多了能复用的脚本 / 配置 / skill。
- **跨工具**：Claude Code 用户读 `CLAUDE.md`，Codex 用户读 `AGENTS.md`，同一套课程两种实现。

## 课程地图

| 层 | 主题 | 你会亲手产出 |
|----|------|-------------|
| 0 | 开课摸底与分班（第一次必做） | 你的专属学习路线 + 进度档案 |
| 1 | CLI 核心与会话（零基础友好） | 命令手册 + headless 脚本 |
| 2 | Harness 配置（Settings / 权限 / Hooks / 输出定制） | 一套 `.claude/settings.json` + hook 脚本 + 自定义 `/命令` |
| 3 | MCP 集成 | 一个你自己写的 toy MCP server |
| 4 | 多 Agent 协同（单实例内） | 编排实验记录 + worktree 分支产物 |
| 5 | 多 AI · 多设备协同 | 多工具协作 pipeline + 交接约定 |
| 6 | Agent SDK | 一个能跑的小型 orchestrator |
| 7 | Skill 工程 + 虚拟项目实战 | 完整虚拟项目 + 可安装的自研 skill/plugin |

## 前置条件

- **必装**：[Claude Code CLI](https://claude.com/claude-code)（或 [Codex CLI](https://github.com/openai/codex)）
- **建议**：Python 3.10+、Node.js 18+（部分实验用到）
- **可选**：Codex / Gemini CLI / Aider（层5 多工具协作会用到，没有也能用模拟方式学）
- **设备**：一台机器即可。涉及"多设备"的内容用云端 agent / 多进程 / 模拟方式完成。

## 快速开始

```bash
git clone https://github.com/CyrusZhang23/AI-learning-by-claude-code.git
cd AI-learning-by-claude-code

# Claude Code 用户：
claude
# 然后在会话里输入：
/lab         # 第一次会自动进入层0 摸底分班；之后从你的进度继续
/labs        # 查看全部课程
/lab 1       # 直接开始第 1 层（已摸底过的话）
```

```bash
# Codex 用户：
codex
# Codex 会自动读取 AGENTS.md；直接说"开始第 1 层实验课"即可。
```

## 它是怎么"带你做"的

仓库根目录的 `CLAUDE.md` 定义了一条**教学协议**：AI 一次只给你一步，你执行完把结果贴回去，AI 检查、纠错、讲原理，确认通过才往下走。`/lab <层号>` 命令会加载 `labs/layer-<层号>-*.md` 教案并按此协议驱动整堂课。

**第一次上课**：AI 会先跑层0，问你几个问题（用过命令行吗？写过代码吗？目标是什么？），据此把你分到零基础 / 有基础 / 进阶三条路线之一，并把进度存进 `workspace/progress.md`（私人文件，不进仓库）。下次回来自动从你的进度继续。

## 语言

课程默认**中文**。需要英文时：

- README 顶部点 **English** 切到 [`README.en.md`](./README.en.md)；
- 课程内容随时在 Claude Code 里用 `/translate`（默认英文，也可指定其他语言）一键翻译当前这一层。

## 目录结构

```
.
├── README.md              # 本文件（中文，给人看）
├── README.en.md           # English version
├── CLAUDE.md              # 教学协议 + 课程索引（给 Claude 看，核心接口）
├── AGENTS.md              # Codex 等价接口
├── LICENSE                # CC-BY-4.0
├── .claude/commands/
│   ├── lab.md             # /lab <N> 启动某层（含首次摸底检测）
│   ├── labs.md            # /labs 列出全部课程
│   └── translate.md       # /translate 一键翻译当前教案
├── labs/                  # 分步教案
│   └── layer-0…7-*.md     # 层0 摸底分班 + 层1–7 实验
└── docs/                  # 参考资料（设计/对照分析）
    ├── codex-adaptation.md
    ├── harness-frameworks.md
    └── skill-projects.md
```

## 许可证

本课程内容以 [CC-BY-4.0](./LICENSE) 开源。你可以自由分享、改编、用于任何目的（含商业），只需署名。

## 致谢

课程设计参考了 Anthropic 官方文档、`skill-creator` / `superpowers` 等开源 skill 项目，以及 LangGraph / CrewAI / OpenHands 等 Agent 框架的 harness 设计。详见 `docs/`。
