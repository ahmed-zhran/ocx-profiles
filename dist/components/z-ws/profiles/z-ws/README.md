# ws — OpenCode Workspace Profile

> **Workspace profile for OpenCode** — a comprehensive agent orchestration environment with planning, building, coding, research, review, and documentation capabilities.

---

## Table of Contents

- [Overview](#overview)
- [Configuration](#configuration)
  - [opencode.jsonc](#opencodejsonc)
  - [ocx.jsonc](#ocxjsonc)
  - [tui.json](#tuijson)
  - [parent-tree.json](#parent-treejson)
  - [package.json](#packagejson)
- [Agents](#agents)
  - [Agent Matrix](#agent-matrix)
  - [Agent Details](#agent-details)
  - [Permission Model](#permission-model)
- [MCP Servers](#mcp-servers)
- [Plugins](#plugins)
  - [NPM Plugins](#npm-plugins)
  - [TUI Plugins](#tui-plugins)
  - [Local Plugin Modules](#local-plugin-modules)
- [Skills](#skills)
- [Commands](#commands)
- [Instructions](#instructions)
- [Delegation Architecture](#delegation-architecture)
- [Philosophy](#philosophy)

---

## Overview

The `ws` profile is a full-featured OpenCode workspace built on the **KDCO registry** (`https://registry.kdco.dev`). It provides a multi-agent orchestration architecture with:

- **7 specialized agents** (plan, build, coder, explore, researcher, scribe, reviewer), each with distinct models, temperatures, reasoning efforts, and permission scopes.
- **7 MCP servers** bridging remote services (context7, exa) and local tools (fast-filesystem, github, duckduckgo, playwright, sequential-thinking).
- **2 NPM plugins** for delegation context persistence and markdown table formatting.
- **2 TUI plugins** for subagent status line and usage dashboard.
- **4 local plugin modules** for background agents, OS notifications, workspace management, and git worktrees.
- **7 skills** covering code philosophy, frontend philosophy, code review, plan protocol/review, and skill/MCP management.
- **1 instruction file** enforcing philosophy loading before code changes.

---

## Configuration

### opencode.jsonc

The main configuration file defining models, agents, MCP servers, plugins, and instructions.

| Field | Value |
|-------|-------|
| `model` | `opencode/big-pickle` |
| `small_model` | `opencode/deepseek-v4-flash-free` |
| `instructions` | `["./tools/philosophy.md"]` |
| `plugin` (npm) | `@tarquinen/opencode-dcp@3.1.3`, `@franlol/opencode-md-table-formatter@0.0.6` |

**MCP Servers:**

| Server | Type | Command / URL |
|--------|------|---------------|
| `context7` | remote | `https://mcp.context7.com/mcp` |
| `exa` | remote | `https://mcp.exa.ai/mcp` |
| `fast-filesystem` | local | `npx -y fast-filesystem-mcp` |
| `github` | local | `gh mcp` |
| `duckduckgo` | local | `npx -y duckduckgo-mcp-server` |
| `playwright` | local | `npx -y @playwright/mcp` |
| `sequential-thinking` | local | `npx -y @modelcontextprotocol/server-sequential-thinking` |

### ocx.jsonc

OCX extension configuration.

| Field | Value |
|-------|-------|
| `renameWindow` | `true` |
| `registries` | `kdco` → `https://registry.kdco.dev` |
| `exclude` | `**/CLAUDE.md`, `**/CONTEXT.md`, `**/.opencode/**`, `**/opencode.jsonc`, `**/opencode.json` |

> **Note:** `**/AGENTS.md` is commented out, meaning AGENTS.md files are **included** (not excluded).

### tui.json

Terminal UI plugin configuration.

| Plugin |
|--------|
| `opencode-subagent-statusline` |
| `opencode-usage-dashboard` |

### parent-tree.json

KDCO profile inheritance tree.

| Field | Value |
|-------|-------|
| `self.registry` | `https://registry.kdco.dev` |
| `self.src` | `kdco/ws` |
| `parents` | `[]` (root profile — no parents) |

### package.json

Local dependencies for plugins and tooling.

| Dependency | Purpose |
|------------|---------|
| `opencode-subagent-statusline` | TUI subagent status bar |
| `opencode-usage-dashboard` | TUI usage metrics dashboard |
| `unique-names-generator` | Generates unique readable names for agents/sessions |
| `zod` | Runtime schema validation |
| `node-notifier` | Cross-platform OS notifications |
| `detect-terminal` | Terminal emulator detection |
| `jsonc-parser` | JSON with Comments parser |

---

## Agents

The profile defines **7 agents**, each with a specific model, temperature, reasoning effort level, and strict permission boundaries.

### Agent Matrix

| Agent | Model | Temp | Reasoning Effort | Role |
|-------|-------|------|------------------|------|
| **plan** | `big-pickle` | 0.3 | high | Architecture design & planning orchestrator |
| **build** | `big-pickle` | 0.3 | medium | Primary build orchestrator — delegates to subagents, never implements directly |
| **coder** | `big-pickle` | 0.2 | high | Technical implementation specialist for writing and modifying code |
| **explore** | `deepseek-v4-flash-free` | 0.2 | low | Fast agent specialized for exploring codebases |
| **researcher** | `nemotron-3-super-free` | 0.4 | high | Knowledge architect for external research and documentation |
| **scribe** | `deepseek-v4-flash-free` | 1.0 | low | Human-facing content specialist for documentation and prose |
| **reviewer** | `nemotron-3-super-free` | 0.1 | high | Expert code reviewer for security, performance, and philosophy compliance |

### Agent Details

#### plan
> **Model:** `opencode/big-pickle` | **Temp:** 0.3 | **Reasoning:** high

Architecture design and planning orchestrator. Delegates to subagents but cannot edit, write, or execute shell commands directly. Manages the development plan and coordinates task distribution.

- **Permissions:** edit:deny, write:deny, bash:deny, task:allow, worktree\_\*:allow, context7\_\*:allow

#### build
> **Model:** `opencode/big-pickle` | **Temp:** 0.3 | **Reasoning:** medium

Primary build orchestrator. Coordinates implementation through delegation to subagents — never implements directly. Executes tasks and manages worktrees.

- **Permissions:** edit:deny, write:deny, bash:deny, task:allow, worktree\_\*:allow, exa\_\*:allow

#### coder
> **Agent file:** `agents/coder.md` | **Mode:** subagent
>
> *"Technical implementation specialist for writing and modifying code"*

The primary code-writing agent. Has full read/write/edit and bash access but is denied access to external MCP tools (context7, exa) and planning/todo read operations.

- **Permissions:** read:allow, write:allow, edit:allow, glob:allow, grep:allow, bash:allow; context7\_\*:deny, exa\_\*:deny, gh\_grep\_\*:deny, plan\_read:deny, todoread:deny

#### explore
> **Agent file:** `agents/explore.md` | **Mode:** subagent
>
> *"Fast agent specialized for exploring codebases"*

Lightweight exploration agent using the flash-free model. Optimized for quick read-only codebase searches with very low temperature and reasoning effort.

- **Permissions:** edit:deny, write:deny, plan\_read:deny, todoread:deny; bash limited to read-only commands (glob, grep, find, cat, rg, ls, bat, less, head, tail, wc, sort, uniq, file, stat, du, diff, tree, which, type, shasum, md5sum, sha256sum, xxd, hexdump, readlink, realpath, dirname, basename, jq, yq, htop, ps, echo, printf, env, hostname, date, cal, uptime, uname, id, whoami, lsof, ss, ip, df, free, nproc, timedatectl, systemctl, journalctl, apt-cache, dpkg, rpm, pkg-config). Bash for interactive commands denied.

#### researcher
> **Agent file:** `agents/researcher.md` | **Mode:** subagent
>
> *"Knowledge architect for external research and documentation"*

External research specialist with access to remote MCP services (context7, exa), web fetching, and GitHub grep. Denied write/edit and most bash commands.

- **Permissions:** context7\_\*:allow, exa\_\*:allow, gh\_grep\_\*:allow, webfetch:allow; write:deny, edit:deny, plan\_read:deny, todoread:deny; bash limited to research tools (curl, wget, gh, dig, nslookup, whois, ping, traceroute, nc, nmap, openssl, python3, pip3, uv, jq, yq, rg, cat, less, head, tail, grep, sort, uniq, wc, echo, printf, date, sleep, which, type, readlink, realpath, dirname, basename).

#### scribe
> **Agent file:** `agents/scribe.md` | **Mode:** subagent
>
> *"Human-facing content specialist for documentation and prose"*

Documentation and prose writer with high temperature (1.0) for creative output. Can read, write, edit, glob, and grep, but has no bash or planning access.

- **Permissions:** edit:allow, glob:allow, read:allow, write:allow; bash:\*:deny, plan\_read:deny, todoread:deny

#### reviewer
> **Agent file:** `agents/reviewer.md` | **Mode:** subagent
>
> *"Expert code reviewer for security, performance, and philosophy compliance"*

Lowest temperature (0.1) for precise, deterministic code review. Has access to plan and delegation read operations, plus git diff/log/show/blame. Allowed `rg` for content search. Denied write/edit.

- **Permissions:** plan\_read:allow, delegation\_read:allow, delegation\_list:allow; edit:deny, write:deny; bash limited to git diff\*, git log\*, git show\*, git blame\*, and rg.

### Permission Model

Each agent has a granular permission set that controls access to:

- **System operations:** `edit`, `write`, `bash`
- **Task delegation:** `task` (plan/build only)
- **Worktree management:** `worktree_*` (plan/build only)
- **External MCP tools:** `context7_*`, `exa_*`, `gh_grep_*`
- **Web fetching:** `webfetch`
- **Internal tools:** `plan_read`, `todoread`, `delegation_read`, `delegation_list`
- **Read-only tools:** `read`, `glob`, `grep` (available to most agents)

Agents that can edit/write code: **coder**, **scribe**
Agents that can run bash: **coder** (full), **explore** (read-only), **researcher** (research tools), **reviewer** (git/rg only)
Agents that can delegate tasks: **plan**, **build**
Agents with external MCP access: **plan** (context7), **build** (exa), **researcher** (context7, exa, GitHub grep, webfetch)

---

## MCP Servers

| Server | Type | Access | Description |
|--------|------|--------|-------------|
| **context7** | remote | plan, researcher | Model Context Protocol server at `mcp.context7.com` |
| **exa** | remote | build, researcher | Model Context Protocol server at `mcp.exa.ai` |
| **fast-filesystem** | local | coder, explore, scribe | Fast filesystem operations via `npx -y fast-filesystem-mcp` |
| **github** | local | coder (partial), researcher, reviewer | GitHub API via `gh mcp` |
| **duckduckgo** | local | researcher | Web search via `npx -y duckduckgo-mcp-server` |
| **playwright** | local | coder | Browser automation via `npx -y @playwright/mcp` |
| **sequential-thinking** | local | plan, build, reviewer | Structured reasoning via `npx -y @modelcontextprotocol/server-sequential-thinking` |

---

## Plugins

### NPM Plugins

| Plugin | Version | Purpose |
|--------|---------|---------|
| `@tarquinen/opencode-dcp` | 3.1.3 | Delegation Context Persistence — maintains context across delegation turns |
| `@franlol/opencode-md-table-formatter` | 0.0.6 | Markdown table formatting and alignment |

### TUI Plugins

| Plugin | Purpose |
|--------|---------|
| `opencode-subagent-statusline` | Displays current subagent status in terminal status line |
| `opencode-usage-dashboard` | Visual dashboard for token/model usage metrics |

### Local Plugin Modules

Located in `plugins/`:

#### background-agents.ts
> **JSDoc:** *"Unified delegation system for OpenCode"*

Replaces the native `task` tool with a persistent, async-first delegation system. Enables background agent execution with status tracking, delegation lifecycle management, and continuity across turns.

#### notify.ts
> **JSDoc:** *"Native OS notifications for OpenCode"*

Sends native OS desktop notifications when the AI requires human attention (e.g., permission requests, task completion, errors). Uses `node-notifier` for cross-platform support.

#### workspace-plugin.ts
> **JSDoc:** *"KDCO Workspace Plugin"* / *"Provides plan management and targeted rule injection"*

Exposes `plan_save` and `plan_read` tools for persistent plan management. Injects routing rules into agent communication for targeted delegation.

#### worktree.ts
> **JSDoc:** *"OCX Worktree Plugin"*

Creates isolated git worktrees for AI development sessions. Each session gets a dedicated worktree with seamless terminal spawning, preventing cross-session contamination.

---

## Skills

Located in `skills/`. Each skill has a `SKILL.md` with frontmatter metadata.

| Skill | Description |
|-------|-------------|
| **code-philosophy** | Internal logic and data flow philosophy — *The 5 Laws of Elegant Defense* (Defensive Depth, Immutable State, Pure Functions, Fail Fast, Comprehensive Contracts) |
| **frontend-philosophy** | Visual and UI philosophy — *The 5 Pillars of Intentional UI* (Intentional State, Z-Axis Awareness, Polymorphic Components, Anticipatory UX, Explicit Gestures) |
| **code-review** | Comprehensive code review methodology with severity classification and confidence thresholds. Systematic approach to security, performance, maintainability, and philosophy compliance |
| **plan-protocol** | Guidelines for creating and managing implementation plans with citation requirements |
| **plan-review** | Criteria for reviewing implementation plans against quality standards (clarity, completeness, feasibility, consistency) |
| **z-mcp** | Discover, search, vet, install, and delete MCP servers from 3 provider indexes |
| **z-skill** | Discover, search, vet, install, and delete agent skills from 4 provider indexes |

---

## Commands

Located in `commands/`.

| Command | File | Description |
|---------|------|-------------|
| **review** | `commands/review.md` | Run code review on files or recent changes. Invokes the code review workflow against loaded philosophy standards |

---

## Instructions

Located in `tools/`.

| File | Purpose |
|------|---------|
| `tools/philosophy.md` | **Code Philosophy — Mandatory.** Before writing or modifying code, agents MUST load the relevant philosophy skill (code-philosophy for backend, frontend-philosophy for UI), verify implementation against the philosophy checklist, and refactor if any principle is violated |

This instruction file is registered in `opencode.jsonc` under the `instructions` field, making it a system-level constraint applied to every agent session.

---

## Delegation Architecture

The `ws` profile enforces a **strict delegation hierarchy**:

```
plan ──► task ──► build ──► task ──► coder/explore/researcher/scribe/reviewer
  │                                 ▲
  │                                 │
  └────── worktree isolation ───────┘
```

1. **plan** designs the architecture and creates worktrees
2. **build** orchestrates implementation through delegation
3. **coder/explore/researcher/scribe/reviewer** execute specific subtasks
4. The **DCP plugin** (`@tarquinen/opencode-dcp`) persists delegation context across turns
5. The **background-agents plugin** replaces the native task tool with async-first delegation

Agents with `task:allow` (plan, build) can delegate; agents without it must be invoked directly or by a delegator.

---

## Philosophy

The profile is governed by two complementary code philosophies, enforced via `tools/philosophy.md`:

### Code Philosophy — The 5 Laws of Elegant Defense
*Loaded for backend/logic tasks*

1. **Defensive Depth** — Validate at every boundary, never trust input
2. **Immutable State** — Prefer `const` and immutable data structures
3. **Pure Functions** — Same input → same output, no side effects
4. **Fail Fast** — Crash early with clear error messages
5. **Comprehensive Contracts** — TypeScript types, runtime validation, documented invariants

### Frontend Philosophy — The 5 Pillars of Intentional UI
*Loaded for UI/frontend tasks*

1. **Intentional State** — Every state must have a purpose
2. **Z-Axis Awareness** — Think in layers, not just pixels
3. **Polymorphic Components** — One component, many contexts
4. **Anticipatory UX** — Design for what the user will do next
5. **Explicit Gestures** — Every interaction must feel intentional

---

## Quick Reference

| Item | Count |
|------|-------|
| Agents | 7 |
| MCP Servers | 7 |
| NPM Plugins | 2 |
| TUI Plugins | 2 |
| Local Plugin Modules | 4 |
| Skills | 7 |
| Commands | 1 |
| Instruction Files | 1 |
| Registry | kdco.dev (OCX) |
| Models | big-pickle (main), deepseek-v4-flash-free (small) |
