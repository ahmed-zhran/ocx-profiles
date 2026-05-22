# Task: Generate README.md for ws profile

You are in an OpenCode profile directory. Create (or update) a comprehensive `README.md` in this directory that documents every aspect of the profile.

## What to do

Read ALL of the following files in the current directory to understand the profile:

1. **`opencode.jsonc`** — Model config (main, small), agent definitions with model assignments, npm plugins, MCP servers, instruction files
2. **`ocx.jsonc`** — Features (renameWindow), exclusion patterns, registries
3. **`tui.json`** — TUI plugin list
4. **`agents/*.md`** — Each agent's description (extract from YAML frontmatter `description:` field) and mode (`mode:` field)
5. **`plugins/*.ts`** — System plugin names. Extract descriptions from the JSDoc block at the top of each file (first meaningful ` * ` line). Skip files under `kdco-primitives/` (they are utilities, not standalone plugins).
6. **`skills/*/SKILL.md`** — Skill names and descriptions from YAML frontmatter
7. **`commands/*.md`** — Command names and descriptions from YAML frontmatter
8. **`tools/*.md`** — Instruction files loaded by the profile
9. **`parent-tree.json`** — Registry provenance tracking info

## Required README Structure

Write `README.md` with these sections in order:

### 1. Title & Tagline
```markdown
# Profile: ws
> Production-grade OpenCode workspace profile with agent orchestration.
```

### 2. Quick Start
```bash
# Add the registry (one-time)
ocx registry add https://ahmed-zhran.github.io/ocx-profiles/ --name zhran --global

# Install this profile
ocx profile add ws --source zhran/ws --global

# Launch OpenCode with this profile
ocx opencode -p ws
```

### 3. Features
From `ocx.jsonc`:
- If `renameWindow` is true, mention that terminal windows are renamed to the active project name
- Explain secure context isolation: project AGENTS.md is included for task awareness, while CLAUDE.md, CONTEXT.md, and .opencode/ directories are excluded to prevent config leakage
- List all exclusion patterns as a sub-list

### 4. Model Configuration
Table with columns: Role | Model
Rows: Main (from `model`), Small (from `small_model`)

### 5. Agents
Two sub-tables. Classify:
- **Orchestration Agents**: `plan` and `build` (they delegate, have `task` permission, edit restricted)
- **Subagents**: `coder`, `explore`, `researcher`, `reviewer`, `scribe` (they perform specific roles)

Each table has columns: Agent | Model | Description
Extract descriptions from `agents/*.md` frontmatter `description:` field. If an agent has no file, check inline `description` in `opencode.jsonc`.

### 6. Orchestration Flow
Describe the agent delegation workflow:
- `build` (primary orchestrator) receives high-level tasks, delegates to specialists, never implements directly
- `plan` handles architecture design and planning
- `coder` implements code with full write/edit/bash permissions, follows philosophy skills, runs verification
- `explore` searches codebase with read-only commands (find, grep, git)
- `researcher` gathers external knowledge from web, GitHub, docs via MCP tools
- `reviewer` reviews code for correctness, security, performance, style; reports by severity
- `scribe` writes documentation, commits, changelogs
- All agents run async via task mechanism, each with own model and permissions

### 7. Plugins
Three sub-sections:

**System Plugins** — Bullet list of `plugins/*.ts` files (skip `kdco-primitives/`). Format: `- **name**: description`

**NPM Plugins** — Inline code list of npm packages from `opencode.jsonc` `plugin` array. Format: `` - `package@version` ``

**TUI Plugins** — Inline code list from `tui.json` `plugin` array. Format: `` - `plugin-name` ``

### 8. MCP Servers
Table with columns: Server | Type | URL
Read from `opencode.jsonc` `mcp` config.

### 9. Skills
Table with columns: Skill | Description
Read from `skills/*/SKILL.md` frontmatter.

### 10. Commands
Table with columns: Command | Description
Read from `commands/*.md` frontmatter.

### 11. Tools / Instructions
List instruction files from `opencode.jsonc` `instructions` array. For each, read the first few lines to determine its purpose.

### 12. Registry Information
- **Registry URL**: https://ahmed-zhran.github.io/ocx-profiles/
- **Component**: zhran/ws
- **Profile Path**: `~/.config/opencode/profiles/ws/`

## Quality Requirements

- EVERY section must be present, even if data is sparse (use "(none)" placeholder)
- Use proper markdown throughout (tables, code fences, bold, lists)
- Agent descriptions must be extracted from frontmatter — not generic placeholders
- Plugin descriptions from JSDoc must be the real one-liner, not the name repeated
- Be accurate: read each file, don't invent data
- The README should be immediately useful to someone who clones this profile

## Output

Write the result to `README.md` in the current directory. Overwrite if it exists.
