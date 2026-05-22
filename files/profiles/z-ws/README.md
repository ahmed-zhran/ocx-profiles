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

- **Terminal Renaming**: Terminal windows are renamed to the active project name (`renameWindow: true`)
- **Secure Context Isolation**: Project `AGENTS.md` is included for task awareness, while `CLAUDE.md`, `CONTEXT.md`, `.opencode/` directories, and `opencode.jsonc/json` files are excluded to prevent configuration leakage between projects
- **Exclusion Patterns**:
  - `**/CLAUDE.md`
  - `**/CONTEXT.md`
  - `**/.opencode/**`
  - `**/opencode.jsonc`
  - `**/opencode.json`

## Model Configuration

| Role  | Model                         |
|-------|-------------------------------|
| Main  | `opencode/big-pickle`         |
| Small | `opencode/deepseek-v4-flash-free` |

## Agents

### Orchestration Agents

| Agent  | Model                 | Description                                                                 |
|--------|-----------------------|-----------------------------------------------------------------------------|
| `plan` | `opencode/big-pickle` | Architecture design and planning specialist. Has `task` permission, edit/write/bash denied. |
| `build` | `opencode/big-pickle` | Primary orchestrator. Coordinates implementation via delegation — never implements directly. Has `task` permission, edit/write/bash denied. |

### Subagents

| Agent       | Model                           | Description                                                                 |
|-------------|---------------------------------|-----------------------------------------------------------------------------|
| `coder`     | `opencode/big-pickle`           | Technical implementation specialist for writing and modifying code          |
| `explore`   | `opencode/deepseek-v4-flash-free` | Codebase search specialist using read-only commands (find, grep, git)       |
| `researcher`| `opencode/nemotron-3-super-free` | Knowledge architect for external research and documentation                 |
| `reviewer`  | `opencode/nemotron-3-super-free` | Expert code reviewer for security, performance, and philosophy compliance   |
| `scribe`    | `opencode/deepseek-v4-flash-free` | Human-facing content specialist for documentation and prose                 |

## Orchestration Flow

All agents run asynchronously via the task/delegation mechanism, each with its own model, permission set, and system prompt.

- **`build`** (primary orchestrator) receives high-level tasks from the user, delegates work to specialist agents, interprets results, and decides next steps. It never implements directly.
- **`plan`** handles architecture design and implementation planning. It delegates research to `researcher`/`explore`, designs the plan, and saves it via `plan_save`.
- **`coder`** implements code with full write/edit/bash permissions. Before implementing, it loads the relevant philosophy skill (`code-philosophy` or `frontend-philosophy`), follows its rules, and runs verification (lint, type-check, tests) after changes.
- **`explore`** searches the codebase using read-only commands (`find`, `grep`, `rg`, `git status/log/diff`, `cat`, `head`, `tail`, `wc`, `tree`, etc.). It cannot access external resources.
- **`researcher`** gathers external knowledge from web searches, GitHub, documentation, and package registries via MCP tools (`context7`, `exa`, `gh_grep`) and read-only bash commands. It returns complete, implementation-ready findings with citations.
- **`reviewer`** reviews code for correctness, security, performance, and style. Findings are classified by severity (🔴 Critical, 🟠 Major, 🟡 Minor, 🟢 Nitpick) and only reported at ≥80% confidence.
- **`scribe`** writes documentation, commit messages, PR descriptions, and changelogs. It can read and write files but cannot execute shell commands.

### Read-only vs Write-capable Routing

| Agent Type | Tool Used | Reason |
|------------|-----------|--------|
| Read-only sub-agents (`explore`, `researcher`, `reviewer`) | `delegate` | Background sessions, async, auto-persisted |
| Write-capable sub-agents (`coder`, `scribe`) | `task` | Native task, preserves undo/branching |

## Plugins

### System Plugins

- **background-agents**: Unified delegation system for OpenCode. Replaces native `task` tool with persistent, async-first agent delegation. All agent outputs are persisted to storage; the orchestrator receives only key references with notification tags.
- **notify**: Native OS notifications for OpenCode. Notifies the human when the AI needs them back (task ready, error, permission required). Uses cmux notifications when available, with desktop notification fallback. Auto-detects terminal emulator and suppresses notifications when terminal is focused.
- **workspace-plugin**: KDCO Workspace Plugin. Provides plan management (`plan_save`, `plan_read`), targeted rule injection (agent routing, philosophy loading, code review protocol), coder task tracking for review triggers, and compaction hooks to inject plan context.
- **worktree**: OCX Worktree Plugin. Creates isolated git worktrees for AI development sessions with seamless terminal spawning across macOS, Windows, and Linux. Supports config file sync, directory symlinking, and post-create/pre-delete hooks.

### NPM Plugins

- `@tarquinen/opencode-dcp@3.1.3`
- `@franlol/opencode-md-table-formatter@0.0.6`

### TUI Plugins

- `opencode-subagent-statusline`
- `opencode-usage-dashboard`

## MCP Servers

| Server   | Type   | URL                           |
|----------|--------|-------------------------------|
| context7 | remote | `https://mcp.context7.com/mcp`  |
| exa      | remote | `https://mcp.exa.ai/mcp`        |
| gh_grep  | remote | `https://mcp.grep.app`          |

## Skills

| Skill               | Description                                                                                                                          |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| code-philosophy     | Internal logic and data flow philosophy (The 5 Laws of Elegant Defense). Understand deeply to ensure code guides data naturally and prevents errors. |
| code-review         | Comprehensive code review methodology with severity classification and confidence thresholds.                                        |
| frontend-philosophy | Visual & UI philosophy (The 5 Pillars of Intentional UI). Understand deeply to avoid "AI slop" and create distinctive, memorable interfaces. |
| plan-protocol       | Guidelines for creating and managing implementation plans with citations.                                                            |
| plan-review         | Criteria for reviewing implementation plans against quality standards.                                                               |

## Commands

| Command  | Description                                            |
|----------|--------------------------------------------------------|
| `review` | Run code review on files or recent changes             |

## Tools / Instructions

| File | Purpose |
|------|---------|
| `./tools/philosophy.md` | **Code Philosophy - MANDATORY**: Before writing or modifying any code, agents must load the relevant philosophy skill (`frontend-philosophy` for UI, `code-philosophy` for backend), verify implementation against the philosophy checklist, and refactor if needed. |

## Registry Information

- **Registry URL**: `https://registry.kdco.dev`
- **Component**: `kdco/ws`
- **Profile Path**: `~/.config/opencode/profiles/ws/`
- **Alias**: Also installable as `zhran/ws` from `https://ahmed-zhran.github.io/ocx-profiles/`
