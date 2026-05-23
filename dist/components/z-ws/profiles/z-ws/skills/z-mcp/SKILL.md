---
name: z-mcp
description: Discover, search, vet, install, and delete MCP servers from 3 provider indexes across 3 OpenCode config targets. Use when the user asks to find, install, remove, or vet MCP servers.
---

# z-mcp — MCP Server Manager

## Triggers

Load this skill when the user says any of the following:
- find MCP / find MCP server
- search MCP / search MCP servers
- install MCP / install MCP server / add MCP server
- remove MCP / delete MCP / uninstall MCP server
- vet MCP / review MCP / audit MCP server
- list MCP servers / show MCP servers / show MCP config
- configure MCP / MCP config
- MCP provider / where to find MCP servers

## 3 MCP Providers (Search & Browse)

| Provider | Type | Search Method | Server Count | Built-in Security |
|---|---|---|---|---|
| mcp.directory | Server directory | Browse + one-click install | 3,000+ | None |
| mcpservers.org | Awesome list mirror | Browse by category | 450+ curated | Community-rated (stars) |
| glama.ai | Full registry + Gateway | Browse/search + in-browser inspect + API | 22,820+ | Quality/safety scores (F-A) |

### 1. mcp.directory

**URL:** https://mcp.directory/servers

**What it is:** An IDE-focused MCP server directory with one-click install for Cursor, VS Code, Claude Desktop, and ChatGPT. Best for users who want to browse visually and install with minimal friction.

**Search:** Browse servers by category at https://mcp.directory/servers. Each server page shows install commands for each supported client.

**Install:** Each server page provides one-click install buttons for different clients. For OpenCode, derive the config from the provided commands. Typical patterns:

```bash
# npx-based (local stdio)
npx -y @modelcontextprotocol/server-git

# pip-based (local stdio)
pip install mcp-server-sqlite && python -m mcp_server_sqlite

# docker-based
docker run -i --rm mcp/filesystem /path/to/allowed/dir
```

### 2. mcpservers.org

**URL:** https://mcpservers.org/

**What it is:** A curated mirror of the punkpeye/awesome-mcp-servers GitHub repo (87.5K stars). Tracks ~450 production-ready MCP servers organized by category. Uses GitHub-native discovery based on stars, release hygiene, and maintainer activity.

**Search:** Browse categories at https://mcpservers.org/. Each entry links to the server's GitHub repo with stars, description, and last update.

**Install:** Each server entry links to its GitHub repo. The install method varies by implementation language:

```bash
# TypeScript / Node.js
npx -y @owner/mcp-server

# Python
pip install mcp-server-name
uvx mcp-server-name

# Go
go install github.com/owner/mcp-server@latest

# Rust
cargo install mcp-server-name

# Docker
docker run -i --rm owner/mcp-server
```

### 3. glama.ai (22,820+ Servers)

**URL:** https://glama.ai/mcp/servers

**What it is:** The largest MCP server registry with quality scoring (F to A grade), safety scores, in-browser testing via Glama MCP Inspector, and hosted deployment. Servers are maintainer-verified and continuously rebuilt.

**Search:**
```bash
# Browse and search at:
https://glama.ai/mcp/servers

# Filter by category (86 curated), quality score, or keyword
# Each server shows: tools list, install command, quality score, license, last update
```

**Install:** Each server page shows the install config for different clients. Glama also provides hosted connectors (remote endpoints) that can be used without local install:

```bash
# Local install (from server page)
npx -y @owner/mcp-server-name

# Hosted connector (remote URL)
# Use the Gateway URL provided on the server page
```

**API access (programmatic search):**
```bash
# Search via Glama API
curl https://glama.ai/api/mcp/v1/servers?q=<query>

# Glama also provides an official MCP registry search server:
# uvx mcp-glama-registry
# Exposes a search_mcp_servers tool
```

## 3 Install/Delete Targets (MCP Config Locations)

| # | Target | File Path | MCP Format |
|---|--------|-----------|------------|
| 1 | Project (local) | `./opencode.json` + `./.mcp.json` | `mcp` + `mcpServers` |
| 2 | OpenCode profile | `~/.config/opencode/profiles/<active-profile>/opencode.jsonc` | `mcp` |
| 3 | Global OpenCode | `~/.config/opencode/opencode.jsonc` | `mcp` |

## OpenCode MCP Config Format

For OpenCode, MCP servers are configured in `opencode.jsonc` under the `mcp` key:

```json
{
  "mcp": {
    "server-name": {
      "type": "local",
      "command": ["npx", "-y", "@package/mcp-server"],
      "enabled": true,
      "env": {
        "API_KEY": "${ENV_VAR_OR_VALUE}"
      }
    }
  }
}
```

**For remote (HTTP) servers:**
```json
{
  "mcp": {
    "server-name": {
      "type": "remote",
      "url": "https://mcp.example.com/sse",
      "enabled": true,
      "headers": {
        "Authorization": "Bearer ${TOKEN}"
      }
    }
  }
}
```

**Notes:**
- `type` is required: `"local"` (stdio process) or `"remote"` (HTTP/S endpoint)
- `command` is an array of strings (NOT a single string)
- `enabled: true` activates the server; `enabled: false` disables inherited servers
- `env` object provides environment variables to local stdio processes
- `headers` object provides HTTP headers for remote connections
- Use `${ENV_VAR}` syntax to reference environment variables (security best practice — never hardcode secrets)

## .mcp.json Universal Config Format

`.mcp.json` at the project root is the emerging universal MCP config standard. It uses the `mcpServers` key and is read natively by Claude Code, Cursor, and other tools.

All targets use the OpenCode-native `mcp` key format.

```json
{
  "mcpServers": {
    "<server-name>": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@package/mcp-server"],
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

## Profile Auto-Detection

When the user wants to configure an MCP server in their OpenCode profile, determine the active profile:

1. **Check `$OPENCODE_PROFILE`** environment variable — if set, use it directly
2. **Check `~/.config/opencode/opencode.jsonc`** — look for a `profile` field
3. **List profiles** — check `~/.config/opencode/profiles/*/`:
   - If only one profile exists, use it
   - If multiple profiles exist, ask the user which one
4. **Fallback** — ask the user: "Which profile would you like to configure?"

## Install Workflow (Add MCP Server to Config)

When the user asks to install/add/configure an MCP server:

### Step 1: Identify the Server

Ask the user what MCP server they want. If they don't know the exact name:
- Search one of the 3 providers for relevant servers
- Present options with: name, description, quality score (Glama), stars (GitHub), transport type

### Step 2: Determine the Config Format

For each server, determine:
- **Transport type**: local stdio or remote HTTP
- **Command/URL**: the npx/uvx/pip command or remote URL
- **Required env vars**: any API keys, tokens, or configuration needed
- **Arguments/flags**: any additional arguments needed

Consult the server's provider page or GitHub README for details.
Also determine which config target is appropriate — project, profile, or global.

### Step 3: Generate Config Snippet

Generate the OpenCode-compatible MCP config entry. Choose the format based on transport type:

**For local (stdio) servers:**
```json
{
    "mcp": {
        "<server-name>": {
            "type": "local",
            "command": ["npx", "-y", "@package/mcp-server"],
            "enabled": true,
            "env": {}
        }
    }
}
```

**For remote (HTTP) servers:**
```json
{
    "mcp": {
        "<server-name>": {
            "type": "remote",
            "url": "https://example.com/mcp",
            "enabled": true,
            "headers": {}
        }
    }
}
```

### Step 4: Ask User for Target Config

Ask: "Where should this MCP server be configured?"

1. **Project (local)** — creates/updates both `./opencode.json` (OpenCode-native `mcp` key) and `./.mcp.json` (cross-agent `mcpServers` key)
2. **OpenCode profile** — `~/.config/opencode/profiles/<active-profile>/opencode.jsonc`
3. **Global OpenCode** — `~/.config/opencode/opencode.jsonc`

### Step 5: Edit Config File

1. Read the target config file(s) (create if they don't exist)
2. Parse existing JSON/JSONC content
3. **For local project target:** add the server under `mcp` in `opencode.json` AND under `mcpServers` in `.mcp.json`
4. **For profile/global targets:** add the server under the `mcp` key
5. If the key doesn't exist yet, create it
6. Write the file(s) back, preserving existing content and formatting

**CRITICAL:** When editing `opencode.jsonc` (JSONC with comments), preserve ALL existing fields, comments, and formatting. Only add or modify the specific MCP server entry.

### Step 6: Notify User

Tell the user: "MCP server `<server-name>` has been added to `<config-file>`. **Please quit and restart OpenCode** for the change to take effect."

## MCP Security Vetting Protocol (BEFORE EVERY INSTALL)

**Run the MCP vetting protocol BEFORE adding any MCP server to a config.**

### Step 1: Source Metadata Check

Gather and present:
- **Provider** — Which directory found this server? (mcp.directory / mcpservers.org / glama.ai)
- **Server name** — The package name or identifier
- **Author/Publisher** — Who created it? What is their reputation?
- **Stars** — GitHub stars or platform rating
- **Quality Score** — Glama grade (F-A) if available
- **Downloads** — npm/pip download count
- **Last Update** — When was it last maintained?
- **Purpose** — What does the server do? What tools/functions does it expose?

### Step 1.5: Provider Quality & Vulnerability Check

Check quality/vulnerability signals from the source providers:

1. **Glama quality score** — Glama grades servers F (poor) to A (excellent). Only recommended option for servers graded C or higher.
   - Query: `curl https://glama.ai/api/mcp/v1/servers?q=<server-name>` — check the `grade` field
2. **mcpservers.org curation** — Servers listed here are manually curated from punkpeye/awesome-mcp-servers. Check GitHub stars and release hygiene on the server's repo.
3. **mcp.directory popularity** — Check install count and user ratings if available.
4. **Known CVEs** — Search `npm audit <package>` or `pip audit <package>` for known vulnerabilities in the server package.
5. **Community reports** — Search the web for `<server-name> mcp security issue` to check for known problems.

**If any provider flags a server as poor quality (Glama D/F), has known CVEs, or has negative community reports, do NOT install. Present findings to the user and recommend an alternative.**

### Step 2: Transport & Data Flow Assessment

Check how the server handles data:

1. **Transport type**:
   - **Local stdio** — runs as a local process on your machine. Data stays local.
   - **Remote HTTP** — sends data over the network to external servers. Data leaves your machine.

2. **Required env vars** — What API keys or tokens are needed? Where do they go?
   - e.g., `GITHUB_TOKEN` → sent to GitHub API (legitimate)
   - e.g., `OPENAI_API_KEY` → sent to OpenAI (legitimate)
   - e.g., `MY_SECRET_KEY` → sent to unknown endpoint (RED FLAG)

3. **File system access** (for local stdio servers):
   - Does the server read local files? Which paths?
   - Does the server write to the file system?

4. **Network access** (for local stdio servers):
   - What external services does it connect to?
   - Is network access documented and necessary for the functionality?

### Step 3: Tool/Function Review

List every tool/function the MCP server exposes. Check for:

| Check | What to look for |
|-------|-----------------|
| Read access | Can it read files, databases, emails? |
| Write access | Can it modify files, send messages, create resources? |
| Execute access | Can it run shell commands? |
| Delete access | Can it delete resources, files, data? |
| Data exfiltration | Does it send data to external endpoints? |
| Credential capture | Does it have access to credentials/tokens? |

### Step 4: Risk Grade

| Grade | Criteria |
|-------|----------|
| **LOW** | Well-known author/publisher, local stdio only, well-documented tools, no unnecessary permissions, Glama A-B grade |
| **MEDIUM** | Remote transport with well-known endpoint, requires API keys for legitimate services, moderate tool permissions |
| **HIGH** | Unknown author, remote transport to unknown endpoint, broad file system access, execute/delete capabilities |
| **EXTREME** | Data exfiltration patterns, unknown remote endpoints with auth tokens, shell execution, obfuscated code |

### Step 5: User Confirmation

Present the full vetting report and explicitly ask whether to proceed:

```markdown
## MCP Server Vetting Report: <server-name>

**Provider:** ...
**Author:** ...
**Stars:** ...
**Quality Score:** ...
**Transport:** local / remote
**Risk Grade:** [LOW | MEDIUM | HIGH | EXTREME]

**Tools Exposed:**
- tool-1 (reads ...)
- tool-2 (writes ...)

**Data Flow:**
- Runs as: local process / remote endpoint
- Sends data to: ...
- Requires env vars: [list]

**Do you want to configure this MCP server? (yes/no)**
```

Wait for user confirmation before editing any config file.

## Delete Workflow (Remove MCP Server from Config)

When the user asks to remove or delete an MCP server:

1. **Identify the target** — determine which config file the server is in:
   - Project (local) — check both `./opencode.json` and `./.mcp.json`
   - Profile config
   - Global config

2. **Read the config file** and find the server entry under `mcp`

3. **Remove the entry** — delete the server object from the config

4. **Write the file back** — preserve all other content and formatting

5. **Confirm** — report what was removed and remind user to restart

### Target Configs for Deletion

| Target | Path |
|--------|------|
| Project (local) | `./opencode.json` + `./.mcp.json` |
| OpenCode profile | `~/.config/opencode/profiles/<active-profile>/opencode.jsonc` |
| Global OpenCode | `~/.config/opencode/opencode.jsonc` |

**Example:**
> User: "Remove the playwright MCP from my profile"
> Action: Read `~/.config/opencode/profiles/ws/opencode.jsonc`, find `mcp.playwright`, delete it, write back
> Response: "MCP server `playwright` has been removed from your ws profile. Please quit and restart OpenCode for the change to take effect."

## List Workflow (Show Configured MCP Servers)

When the user asks to list or show MCP servers, check all config targets:

1. **Project (local)** — read `./opencode.json` (under `mcp`) and `./.mcp.json` (under `mcpServers`) if they exist, merge results
2. **OpenCode profile config** — read `~/.config/opencode/profiles/<active-profile>/opencode.jsonc`, extract entries under `mcp`
3. **Global OpenCode config** — read `~/.config/opencode/opencode.jsonc` if it exists, extract entries under `mcp`
4. **Merge and display** all results in a combined table:

| Source | Server Name | Type | Endpoint/Command | Enabled |
|--------|-------------|------|------------------|---------|
| project | context7 | remote | https://mcp.context7.com/mcp | ✅ |
| ws profile | exa | remote | https://mcp.exa.ai/mcp | ✅ |
| global | [none] | - | - | - |

## Update Workflow

When the user asks to update MCP servers:

### For local stdio servers (npx-based):
npx auto-pulls the latest version, but the config stays the same. No action needed unless the server's command/args change.

### For pip/uvx servers:
```bash
pip install --upgrade <package>
# or
uv tool upgrade <package>
```

### For Docker-based servers:
```bash
docker pull <image>
```

### For remote HTTP servers:
No local update needed — the server endpoint is managed by the provider. Just restart OpenCode if the server was recently updated.

### General recommendation:
Tell the user: "MCP servers are typically updated automatically when using `npx` (which always pulls latest) or are managed by the provider (remote servers). To be safe, restart OpenCode to ensure the latest tool definitions are loaded."
