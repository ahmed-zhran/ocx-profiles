# Profile: ws

> Production-grade OpenCode workspace profile with agent orchestration, isolated git worktrees, and secure context management.

A profile for the [ahmed-zhran/ocx-profiles](https://ahmed-zhran.github.io/ocx-profiles/) registry.

---

## Quick Start

```bash
# Add the registry (one-time)
ocx registry add https://ahmed-zhran.github.io/ocx-profiles/ --name zhran --global

# Install this profile
ocx profile add ws --source zhran/ws --global

# Launch OpenCode with this profile
ocx opencode -p ws
```

---

## Features

- **Terminal Window Renaming**: Terminal windows are renamed to the active project name for easy identification.
- **Secure Context Isolation**: Project `AGENTS.md` files are included for task awareness, while `CLAUDE.md`, `CONTEXT.md`, and `.opencode/` directories are excluded to prevent configuration leakage.
- **Operational Exclusions** (from `ocx.jsonc`):
  - `**/CLAUDE.md`
  - `**/CONTEXT.md`
  - `**/.opencode/**`
  - `**/opencode.jsonc`
  - `**/opencode.json`

---

## Model Configuration

| Role  | Model                          |
|-------|--------------------------------|
| Main  | `opencode/big-pickle`          |
| Small | `opencode/deepseek-v4-flash-free` |

---

## Agents

### Orchestration Agents

These agents coordinate work through delegation — they never implement directly.

| Agent  | Model                     | Description |
|--------|---------------------------|-------------|
| `plan` | `opencode/big-pickle`     | Architecture and planning orchestrator — designs implementation plans, coordinates architectural decisions, and manages plan lifecycle via `plan_save`. |
| `build` | `opencode/big-pickle`    | Primary build orchestrator — receives high-level tasks, delegates to specialists (`coder`, `explore`, `researcher`, `scribe`), and interprets results to determine next actions. |

### Subagents

These agents perform specific roles with specialized permissions.

| Agent       | Model                           | Description |
|-------------|---------------------------------|-------------|
| `coder`     | `opencode/big-pickle`           | Technical implementation specialist for writing and modifying code. Follows philosophy skills (code-philosophy / frontend-philosophy) and runs verification after changes. |
| `explore`   | `opencode/deepseek-v4-flash-free` | Codebase exploration specialist — searches files, understands code structure, traces logic. INTERNAL ONLY: cannot access external resources. Read-only. |
| `researcher`| `opencode/nemotron-3-super-free` | Knowledge architect for external research and documentation. Gathers implementation-ready findings from web, GitHub, and documentation via MCP tools. |
| `reviewer`  | `opencode/nemotron-3-super-free` | Expert code reviewer for security, performance, and philosophy compliance. Applies 4 Review Layers (Correctness, Security, Performance, Style) with severity classification. |
| `scribe`    | `opencode/deepseek-v4-flash-free` | Human-facing content specialist for documentation, commit messages, PR descriptions, changelogs, and prose. |

---

## Orchestration Flow

The agent delegation workflow follows a strict orchestration pattern:

1. **`build`** (primary orchestrator) receives high-level tasks from the user, delegates to specialists, and never implements directly.
2. **`plan`** handles architecture design and planning — creates structured implementation plans with citations from research.
3. **`coder`** implements code with full write/edit/bash permissions, follows philosophy skills, and runs verification (lint, type-check, test).
4. **`explore`** searches the codebase with read-only commands (`find`, `grep`, `git log`, `git diff`, etc.).
5. **`researcher`** gathers external knowledge from web, GitHub, documentation, and package registries via MCP tools.
6. **`reviewer`** reviews code for correctness, security, performance, and style; reports findings by severity (🔴 Critical, 🟠 Major, 🟡 Minor, 🟢 Nitpick).
7. **`scribe`** writes documentation, commit messages, PR descriptions, and changelogs.
8. All agents run async via the `task`/`delegate` mechanism, each with their own model and permissions.
9. Background agent outputs are persisted to disk with auto-generated metadata titles/descriptions.
10. Upon completion, orchestrators receive notifications with status summaries and artifact paths for retrieval.

---

## Plugins

### System Plugins

- **background-agents**: Unified delegation system for OpenCode — replaces native `task` tool with persistent, async-first agent delegation. All agent outputs are persisted to storage; orchestrator receives only key references.
- **notify**: Native OS notifications for OpenCode — notifies the human when the AI needs them back, not for every micro-event. Supports cmux, terminal auto-detection, quiet hours, and deduplication.
- **workspace-plugin**: KDCO Workspace Plugin — provides plan management (`plan_save`, `plan_read`) with Zod validation, targeted system-rule injection (`plan`/`build` agent modes), coder task tracking for review triggers, and compaction hooks for plan context persistence.
- **worktree**: OCX Worktree Plugin — creates isolated git worktrees for AI development sessions with seamless terminal spawning across macOS, Windows, and Linux. Supports config file syncing, symlinks, and hooks.

### NPM Plugins

- `@tarquinen/opencode-dcp@3.1.3`
- `@franlol/opencode-md-table-formatter@0.0.6`

### TUI Plugins

- `opencode-subagent-statusline`
- `opencode-usage-dashboard`

---

## MCP Servers

| Server               | Type   | URL / Command                                                    |
|----------------------|--------|------------------------------------------------------------------|
| `context7`           | remote | `https://mcp.context7.com/mcp`                                   |
| `exa`                | remote | `https://mcp.exa.ai/mcp`                                         |
| `filesystem`         | local  | `npx -y @modelcontextprotocol/server-filesystem /home/zhran`     |
| `github`             | local  | `gh mcp`                                                         |
| `duckduckgo`         | local  | `npx -y duckduckgo-mcp-server`                                   |
| `playwright`         | local  | `npx -y @playwright/mcp`                                         |
| `sequential-thinking`| local  | `npx -y @modelcontextprotocol/server-sequential-thinking`        |

---

## Skills

| Skill                | Description |
|----------------------|-------------|
| `code-philosophy`    | Internal logic and data flow philosophy (The 5 Laws of Elegant Defense). Ensures code guides data naturally and prevents errors. |
| `code-review`        | Comprehensive code review methodology with severity classification and confidence thresholds. |
| `frontend-philosophy`| Visual & UI philosophy (The 5 Pillars of Intentional UI). Avoids "AI slop" by creating distinctive, memorable interfaces. |
| `plan-protocol`      | Guidelines for creating and managing implementation plans with citations (`ref:delegation-id`). |
| `plan-review`        | Criteria for reviewing implementation plans against quality standards. |
| `z-mcp`             | Discover, search, vet, install, and delete MCP servers from 3 provider indexes across 3 OpenCode config targets. |
| `z-skill`           | Discover, search, vet, install, and delete agent skills from 4 provider indexes across 4 installation targets. |

---

## Commands

| Command    | Description |
|------------|-------------|
| `review`   | Run code review on files or recent changes. Delegates to the `reviewer` agent with scope from arguments. |

---

## Tools / Instructions

| File                          | Purpose |
|-------------------------------|---------|
| `./tools/philosophy.md`       | Mandates loading the relevant philosophy skill (`code-philosophy` or `frontend-philosophy`) **before** any implementation, verifying against the philosophy checklist before completing, and refactoring if code violates any principle. |

---

## Registry Information

- **Published under**: [ahmed-zhran/ocx-profiles](https://ahmed-zhran.github.io/ocx-profiles/) registry
- **Component**: `zhran/ws`
- **Profile Path**: `~/.config/opencode/profiles/ws/`
- **Provenance**: This profile is published independently under the zhran registry. It was originally derived from `kdco/ws` (registry: `https://registry.kdco.dev`). For the full provenance chain, see `parent-tree.json` in the profile root.
