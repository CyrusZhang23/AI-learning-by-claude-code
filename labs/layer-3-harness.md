# 层3 — Harness 配置

> 可移植性：Settings/Permissions **高**，Hooks **中低**（Codex 差异见 `docs/codex-adaptation.md`）

## 目标

掌控 Claude Code 的"外壳"（harness）：配置的三层覆盖、权限边界、生命周期 hooks、以及输出与命令定制。学完你能把 Claude Code 调成符合自己安全和工作流要求的样子。

## 你将亲手产出

- 一套 `.claude/settings.json`（项目级）+ 一条 `settings.local.json`（本地覆盖）
- 一个 hook 脚本组合：PreToolUse 拦截 + PostToolUse 审计日志 + Stop 通知
- 一个你自己的自定义 slash 命令 `.claude/commands/<你的命令>.md`

## 前置检查

- [ ] 知道 `~/.claude/`（用户级）和项目 `.claude/`（项目级）两个位置
- [ ] 建好 `workspace/lab-02/` 工作区
- [ ] （hook 步骤用）能写简单 shell 脚本

## 分步教案

### 2A — Settings 三层覆盖

#### 步骤 1 — 观察覆盖链
- **做什么**：用 `/config` 看当前生效配置；在项目 `.claude/settings.json` 里设一个键（如 `"model"`），再在 `.claude/settings.local.json` 里设不同值，观察谁赢。
- **为什么**：理解 user → project → local 的优先级，谁覆盖谁。
- **通过标准**：你能预测并验证"同一个键在三处设不同值时最终生效的是哪个"。

### 2B — 权限边界

#### 步骤 2 — allow / deny / ask 规则
- **做什么**：在 settings 的 `permissions` 里加规则：`allow` 放行某类只读命令、`deny` 禁止危险命令（如 `Bash(rm -rf:*)`）、`ask` 对某操作弹确认。然后故意触发一次被 deny 的操作，观察行为。
- **为什么**：权限是 Claude Code 的安全核心——specifier 规则决定哪些工具调用自动放行、拦截、或询问。
- **通过标准**：被 `deny` 的命令确实被挡下；你能写出一条针对特定工具+参数的 specifier。

### 2C — Hooks 生命周期

#### 步骤 3 — PostToolUse 审计日志
- **做什么**：在 settings 配一个 PostToolUse hook，命令是把工具名和时间追加到 `workspace/lab-02/audit.log`。触发几次工具调用，看日志。
- **为什么**：PostToolUse 在工具执行后触发，最适合审计/记录。
- **通过标准**：每次工具调用后 `audit.log` 都新增一行。

#### 步骤 4 — PreToolUse 拦截
- **做什么**：配一个 PreToolUse hook，匹配 Bash 且参数含 `rm -rf` 时返回非零退出码以**阻断**该调用。测试它确实拦截。
- **为什么**：PreToolUse 在工具执行**前**触发，可以否决——这是比权限规则更灵活的防御点。
- **通过标准**：含 `rm -rf` 的 Bash 调用被 hook 拦下，普通命令不受影响。

#### 步骤 5 — Stop 通知
- **做什么**：配一个 Stop hook，在 Claude 结束回合时发一个系统通知（macOS 用 `osascript -e 'display notification ...'`）。
- **为什么**：理解 Stop / SubagentStop 这类"回合结束"事件的用途（提醒、收尾、触发下游）。
- **通过标准**：一次回合结束后你收到了通知。

### 2D — 输出与命令定制

#### 步骤 6 — 自定义 slash 命令
- **做什么**：在 `.claude/commands/` 写一个你自己的命令（如 `review.md`，内容是"审查当前 git diff 并列出潜在 bug"），用 `/review` 触发。了解 `$ARGUMENTS` 参数注入。
- **为什么**：自定义命令 = 把常用 prompt 固化成可复用入口。本仓库的 `/lab` 就是这么做的。
- **通过标准**：`/你的命令` 能正常触发并产出预期结果；带参数的版本能正确读到 `$ARGUMENTS`。

#### 步骤 7 —（可选）output-style / statusline
- **做什么**：看 `/output-style`，了解如何定制回答风格；了解 statusline 如何显示自定义状态行。
- **为什么**：补全"输出层"的定制能力。
- **通过标准**：你能说出 output-style 和 statusline 各改变了什么。

## 完成标志

- settings 三层覆盖你能亲手演示一次
- 三个 hook（拦截/审计/通知）都生效
- 你有了一个能用的自定义 slash 命令
- 你能讲清"权限规则 vs PreToolUse hook"的区别与各自适用场景

## 延伸（可选）

- 用 `hookify` 插件可视化配置 hooks，对照手写 JSON 的差异
- 设计一套"开发/审计"两套 hook profile，用环境变量切换
