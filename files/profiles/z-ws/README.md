# Profile: ws
> Production-grade OpenCode workspace profile with agent orchestration.

## Quick Start

```bash
# Add the registry (one-time)
ocx registry add https://ahmed-zhran.github.io/ocx-profiles/ --name zhran --global

# Install this profile
ocx profile add ws --source zhran/ws --global

# Launch OpenCode with this profile
ocx opencode -p ws
```

## Features

- **Terminal Window Renaming**: Terminal windows are renamed to the active project name for easy identification.
- **Secure Context Isolation**: Project `AGENTS.md` files are included for task awareness (commented-out exclusion), while `CLAUDE.md`, `CONTEXT.md`, `.opencode/` directories, and `opencode.jsonc`/`opencode.json` config files are excluded to prevent configuration leakage between projects.
- **Exclusion Patterns**:
  - `**/CLAUDE.md`
  - `**/CONTEXT.md`
  - `**/.opencode/**`
  - `**/opencode.jsonc`
  - `**/opencode.json`

## Model Configuration

| Role  | Model                        |
|-------|------------------------------|
| Main  | `opencode/big-pickle`        |
| Small | `opencode/deepseek-v4-flash-free` |

## Agents

### Orchestration Agents

These agents delegate work and have `task` permission with edit restrictions.

| Agent  | Model                        | Description                                                |
|--------|------------------------------|------------------------------------------------------------|
| `plan` | `opencode/big-pickle`        | Architecture design and planning orchestrator              |
| `build`| `opencode/big-pickle`        | Primary build orchestrator; delegates implementation, never implements directly |

### Subagents

These agents perform specific implementation, research, and content roles.

| Agent       | Model                           | Description                                                               |
|-------------|---------------------------------|---------------------------------------------------------------------------|
| `coder`     | `opencode/big-pickle`           | Technical implementation specialist for writing and modifying code        |
| `explore`   | `opencode/deepseek-v4-flash-free` | Codebase exploration and analysis agent                                   |
| `researcher`| `opencode/nemotron-3-super-free`  | Knowledge architect for external research and documentation               |
| `reviewer`  | `opencode/nemotron-3-super-free`  | Expert code reviewer for security, performance, and philosophy compliance |
| `scribe`    | `opencode/deepseek-v4-flash-free` | Human-facing content specialist for documentation and prose               |

## Orchestration Flow

The `ws` profile implements a strict delegation-based orchestration workflow:

1. **`build`** (primary orchestrator) receives high-level tasks, delegates to specialists, never implements directly.
2. **`plan`** handles architecture design and planning with meticulous documentation.
3. **`coder`** implements code with full write/edit/bash permissions, follows philosophy skills (code-philosophy or frontend-philosophy), runs verification (lint, type-check, tests).
4. **`explore`** searches the codebase with read-only commands (`find`, `grep`, `rg`, `git log`, etc.).
5. **`researcher`** gathers external knowledge from web, GitHub, docs via MCP tools (`context7`, `exa`, `gh_grep`).
6. **`reviewer`** reviews code for correctness, security, performance, style; reports findings by severity (🔴 Critical, 🟠 Major, 🟡 Minor, 🟢 Nitpick).
7. **`scribe`** writes documentation, commit messages, PR descriptions, changelogs.

All agents run asynchronously via the `delegate`/`task` mechanism, each with its own model assignment and permissions. The `background-agents` plugin provides persistent delegation with progress tracking and notifications.

## Plugins

### System Plugins

- **background-agents**: Unified delegation system for OpenCode — replaces native `task` tool with persistent, async-first agent delegation. All agent outputs are persisted to storage, orchestrator receives only key references.
- **workspace-plugin**: Provides plan management (`plan_save`, `plan_read`), targeted system prompt injection for orchestrator routing, code review protocol, and compaction hooks for plan context retention.
- **notify**: Native OS notifications for OpenCode — sends desktop notifications for session idle, errors, and permission requests. Supports cmux notifications, OSC title updates, quiet hours, and terminal focus detection.
- **worktree**: Creates isolated git worktrees for AI development sessions with seamless terminal spawning across macOS, Windows, and Linux. Includes file sync, symlink support, and configurable hooks.

### NPM Plugins

- `@tarquinen/opencode-dcp@3.1.3`
- `@franlol/opencode-md-table-formatter@0.0.6`

### TUI Plugins

- `opencode-subagent-statusline`
- `opencode-usage-dashboard`

## MCP Servers

| Server   | Type   | URL                          |
|----------|--------|------------------------------|
| context7 | remote | `https://mcp.context7.com/mcp` |
| exa      | remote | `https://mcp.exa.ai/mcp`     |
| gh_grep  | remote | `https://mcp.grep.app`       |

## Skills

| Skill                | Description                                                                                              |
|----------------------|----------------------------------------------------------------------------------------------------------|
| code-philosophy      | Internal logic and data flow philosophy (The 5 Laws of Elegant Defense). Understand deeply to ensure code guides data naturally and prevents errors. |
| code-review          | Comprehensive code review methodology with severity classification and confidence thresholds             |
| frontend-philosophy  | Visual & UI philosophy (The 5 Pillars of Intentional UI). Understand deeply to avoid "AI slop" and create distinctive, memorable interfaces. |
| plan-protocol        | Guidelines for creating and managing implementation plans with citations                                 |
| plan-review          | Criteria for reviewing implementation plans against quality standards                                    |

## Commands

| Command  | Description                                            |
|----------|--------------------------------------------------------|
| `review` | Run code review on files or recent changes             |

## Tools / Instructions

- **`tools/philosophy.md`** — Mandatory code philosophy loading instruction. Requires loading `code-philosophy` or `frontend-philosophy` skills before implementation, verifying against the philosophy checklist, and refactoring if violations exist.

## Registry Information

- **Registry URL**: `https://ahmed-zhran.github.io/ocx-profiles/`
- **Component**: `zhran/ws`
- **Profile Path**: `~/.config/opencode/profiles/ws/`
- **Parent Registry**: `https://registry.kdco.dev` (source: `kdco/ws`)
