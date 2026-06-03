# AI-learning-by-claude-code

[简体中文](./README.md) | **English**

> A **hands-on** course for mastering AI coding tools. Clone it, open Claude Code inside the repo, type `/lab`, and an AI coach will **walk you through every step**. Your first session starts with a quick placement quiz; after that, you run every feature yourself instead of just reading docs.

Focused on **AI coding tools** (Claude Code as the main line, with lateral comparison to Codex / Gemini CLI / Cursor / Aider), the course has 8 layers (0–7) that take you from CLI basics through Harness config, MCP, multi-agent orchestration, multi-AI collaboration, the Agent SDK, and Skill engineering.

## What makes this course different

- **Beginner-friendly**: It assumes you've only used web-based AI and never touched a command line. Your first session (Layer 0) asks a few questions to place you on the right track (beginner / some experience / advanced), then paces the rest to your level.
- **Coaching, not lecturing**: You do every step yourself (run commands, write files); the AI checks your result, explains the why, and only moves on once you've got it.
- **Clone and go**: The teaching interface ships with the repo (`CLAUDE.md` + the `/lab` command) — no extra setup.
- **Every layer produces an artifact**: You finish each layer with reusable scripts / configs / skills, not just "I read it."
- **Cross-tool**: Claude Code users read `CLAUDE.md`, Codex users read `AGENTS.md` — same course, two implementations.

## Course map

| Layer | Topic | What you'll build |
|-------|-------|-------------------|
| 0 | Placement & onboarding (first time only) | Your personalized track + a progress file |
| 1 | CLI core & sessions (beginner-friendly) | Command cheatsheet + headless scripts |
| 2 | Harness config (settings / permissions / hooks / output) | A `.claude/settings.json` + hook scripts + a custom `/command` |
| 3 | MCP integration | A toy MCP server you write yourself |
| 4 | Multi-agent orchestration (within one instance) | Orchestration log + a worktree branch artifact |
| 5 | Multi-AI · multi-device collaboration | A multi-tool pipeline + a handoff contract |
| 6 | Agent SDK | A small working orchestrator |
| 7 | Skill engineering + capstone project | A full toy project + an installable custom skill/plugin |

## Prerequisites

- **Required**: [Claude Code CLI](https://claude.com/claude-code) (or [Codex CLI](https://github.com/openai/codex))
- **Recommended**: Python 3.10+, Node.js 18+ (used in some labs)
- **Optional**: Codex / Gemini CLI / Aider (used in Layer 5 multi-tool collaboration; you can still learn the concepts via simulation without them)
- **Devices**: One machine is enough. "Multi-device" content is done via cloud agents / multiple processes / simulation.

## Quick start

```bash
git clone https://github.com/CyrusZhang23/AI-learning-by-claude-code.git
cd AI-learning-by-claude-code

# Claude Code users:
claude
# Then, inside the session:
/lab         # First run goes to Layer 0 (placement); later, resumes from your progress
/labs        # List all layers
/lab 1       # Jump straight into Layer 1 (once you've been placed)
```

```bash
# Codex users:
codex
# Codex reads AGENTS.md automatically; just say "let's start with Layer 0 placement."
```

## How it "walks you through"

The root `CLAUDE.md` defines a **teaching protocol**: the AI gives you one step at a time, you run it and paste the result back, the AI checks, corrects, explains the underlying mechanism, and only then proceeds. The `/lab <N>` command loads `labs/layer-<N>-*.md` and drives the whole lesson by this protocol.

**First session**: the AI runs Layer 0 first — a few questions (Have you used a command line? Have you coded? What's your goal?) — then places you on one of three tracks (beginner / some experience / advanced) and saves progress to `workspace/progress.md` (a private file, not committed). Next time it resumes from where you left off.

## Language

The course defaults to **Chinese**. For English:

- Click **English** at the top of the README to switch to this file;
- Inside Claude Code, use `/translate` anytime to one-click-translate the current layer (defaults to English; other languages supported too).

## Repository layout

```
.
├── README.md              # Chinese (default)
├── README.en.md           # This file
├── CLAUDE.md              # Teaching protocol + course index (the core interface for Claude)
├── AGENTS.md              # Codex-equivalent interface
├── LICENSE                # CC-BY-4.0
├── .claude/commands/
│   ├── lab.md             # /lab <N> starts a layer (with first-time placement)
│   ├── labs.md            # /labs lists all layers
│   └── translate.md       # /translate one-click translation of the current lesson
├── labs/                  # Step-by-step lesson scripts
│   └── layer-0…7-*.md     # Layer 0 placement + Layers 1–7 labs
└── docs/                  # Reference material (design / comparative analysis)
    ├── codex-adaptation.md
    ├── harness-frameworks.md
    └── skill-projects.md
```

## License

This course content is open-sourced under [CC-BY-4.0](./LICENSE). You're free to share, adapt, and use it for any purpose (including commercial) — just give attribution.

## Acknowledgements

The curriculum draws on Anthropic's official docs, open-source skill projects like `skill-creator` and `superpowers`, and the harness designs of agent frameworks like LangGraph / CrewAI / OpenHands. See `docs/` for details.
