# 层3 — Harness 配置

> 可移植性：Settings/Permissions **高**，Hooks **中低**（Codex 差异见 `docs/codex-adaptation.md`）

> 🎯 **本层一句话**：harness 三件套 —— settings 三层覆盖（local > project > user）、permissions 黑白名单、hooks 任意脚本；hook 不靠环境变量，**靠 stdin 的 JSON 拿上下文、靠退出码表态**（PreToolUse `exit 2` = 否决；其他非零只是非阻断错误）。

## 目标

掌控 Claude Code 的"外壳"（harness）：配置的三层覆盖、权限边界、生命周期 hooks、以及输出与命令定制。学完你能把 Claude Code 调成符合自己安全和工作流要求的样子。

## 你将亲手产出

- 一套 `.claude/settings.json`（项目级）+ 一条 `settings.local.json`（本地覆盖）
- 一个 hook 脚本组合：PreToolUse 拦截 + PostToolUse 审计日志 + Stop 通知
- 一个你自己的自定义 slash 命令 `.claude/commands/<你的命令>.md`

## 前置检查

- [ ] 知道 `~/.claude/`（用户级）和项目 `.claude/`（项目级）两个位置
- [ ] 建好 `workspace/lab-03/` 工作区
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

> **先搞懂 hook 是什么（不理解就别往下做）**
>
> 一句话：**hook 是你在 Claude 工作流程的特定时刻"插一脚"执行自己脚本的机会。**
>
> Claude 跑一轮任务像一条流水线：`你发消息 → Claude 想 → 调工具 → 工具跑完 → 回答 → 回合结束`。默认你只能看不能插手。**hook 就是这条线上预先钻好的几个孔**，每个孔对应一个时刻；你往孔里塞一段 shell 脚本，流水线走到那个时刻就自动停下来先跑你的脚本，跑完再继续。
>
> | 孔的位置 | 事件名 | 你能干什么 |
> |---|---|---|
> | 你的输入刚提交 | `UserPromptSubmit` | 改写/检查 prompt |
> | 要调工具、还没调 | `PreToolUse` | **拦住它**（`exit 2`=取消） |
> | 工具刚跑完 | `PostToolUse` | 记日志/审计 |
> | 整个回合结束 | `Stop` | 发通知 |
>
> **三个本质特征：**
> 1. **自动触发**——配在 `settings.json` 里一次，之后每次到点 harness 自己调，不用你喊。
> 2. **就是普通 shell 脚本**——`command` 就是一行 bash，没有魔法；能跑 `jq`/`osascript`/你自己的 python。
> 3. **靠 stdin 拿上下文 + 靠退出码表态**——harness 把"现在发生了什么"以 **JSON 从 stdin** 喂进来（工具名/参数/输出都在里面）；你用退出码回应（**PreToolUse 只有 `exit 2` 才否决**；`exit 0` 放行，其他非零是非阻断错误，只把 stderr 显示给你、不挡工具）。
>
> **为什么需要它**：权限 `deny` 只能黑名单字符串匹配，死板；hook 是任意脚本，能读参数内容、查文件再决定——这是 Claude Code 可定制性的核心。

#### 步骤 3 — PostToolUse 审计日志
- **做什么**：在 settings 配一个 PostToolUse hook，命令是把工具名和时间追加到 `audit.log`。触发几次工具调用，看日志。
- **为什么**：PostToolUse 在工具执行后触发，最适合审计/记录。
- **通过标准**：每次工具调用后 `audit.log` 都新增一行，**且工具名不为空**。

可用的正确写法（matcher 空串 = 匹配所有工具）：
```json
"PostToolUse": [
  {
    "matcher": "",
    "hooks": [
      { "type": "command",
        "command": "echo \"$(date '+%H:%M:%S') $(cat | jq -r '.tool_name')\" >> ~/tmp/lab2-sandbox/audit.log" }
    ]
  }
]
```

> **两个真实踩过的坑（▶ 动手前先让学员预测）**
>
> 配 hook 前先抛两个问题给学员，让他先猜、再揭晓——这两个坑都是直觉会答错的：
>
> **预测题 1**："hook 脚本怎么知道刚才调的是哪个工具？你觉得工具名是放在某个环境变量里（比如 `$CLAUDE_TOOL_NAME`），还是别的渠道？"
> → 揭晓：**不是环境变量**。Claude Code **不用环境变量传上下文**——所有 hook 数据走 **stdin JSON**。正确做法是 `cat | jq -r '.tool_name'`。PostToolUse 的 payload 字段：`tool_name` / `tool_input` / `tool_response`。
>
> **预测题 2**："如果我先 `INPUT=$(cat)` 把 stdin 存进变量，再 `echo $INPUT | jq ...`，能正常解析吗？" 多数人答"能"。
> → 揭晓：**会失败**。工具输出（如 `ls` 的多行结果）带换行，`echo $INPUT`（无引号）把换行折叠成空格，JSON 结构被打碎，jq 解析出 null。**直接 `cat | jq` 流式处理 stdin**，不经过变量，就没这问题。（若一定要存变量，必须 `printf '%s' "$INPUT" | jq` 并全程加引号。）

#### 步骤 4 — PreToolUse 拦截
- **做什么**：配一个 PreToolUse hook，匹配 Bash 且参数含 `rm -rf` 时以 `exit 2` **阻断**该调用（注意：必须是 `exit 2`，换成 `exit 1` 或其他非零不会拦下来）。测试它确实拦截。
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

### 步骤 8 — 全景扫描（收尾）[核心]
- **做什么**：教练带学员对照本层重点的**六个工具/回合类触发点**（`UserPromptSubmit` / `PreToolUse` / `PostToolUse` / `Notification` / `Stop` / `SubagentStop`），逐个问"这层我们配过哪几个？剩下的是干啥的？"——本层只动手了 4 个，`Notification` 和 `SubagentStop` 没碰，明确它们存在、各自时机。**注意这六个不是全集**：还有 `PreCompact`（压缩前）、`SessionStart` / `SessionEnd`（会话起止）等事件本层完全没碰，扫一眼知道它们存在即可。再看 `settings.json` 里还有哪些没用到的顶层键（如 `env`、`statusLine`）。
- **为什么**：建立"hook 事件全貌 + settings 全貌"的索引感——本层只动手了常用六个里的四个，而 hook 事件还不止这六个。

> **🧪 留个回归夹具**：把本层的 `.claude/settings.json`（三个 hook + permission 规则）连同一行验证命令留在 sandbox 里——以后改教案 / 升级版本时，重跑一次就能确认 hook 仍按预期触发（PostToolUse 写日志、PreToolUse 挡 `rm -rf`）。这是"教案可回归"的最小单位。

## 完成标志

- settings 三层覆盖你能亲手演示一次
- 三个 hook（拦截/审计/通知）都生效
- 你有了一个能用的自定义 slash 命令
- 你能讲清"权限规则 vs PreToolUse hook"的区别与各自适用场景
- 🎯 能复述本层那句话：harness 三件套 + "hook 靠 stdin JSON 拿上下文、靠退出码表态"

## 延伸（可选）

- 用 `hookify` 插件可视化配置 hooks，对照手写 JSON 的差异
- 设计一套"开发/审计"两套 hook profile，用环境变量切换
