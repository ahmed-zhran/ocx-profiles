---
name: z-skill
description: Discover, search, vet, install, and delete agent skills from 4 provider indexes across 4 installation targets. Use when the user asks to find, install, remove, or vet agent skills.
---

# z-skill — Agent Skill Manager

## Triggers

Load this skill when the user says any of the following:
- find a skill / find skills
- search skills / search for a skill
- install skill / install a skill
- remove skill / delete skill / uninstall skill
- vet skill / review skill / audit skill
- list skills / show skills
- skill provider / where to find skills

## 4 Providers (Search & Install)

| Provider | Type | Search Method | Install Method | Skills Count | Built-in Security |
|---|---|---|---|---|---|
| skills.sh | Universal registry | `npx skills find` / browse | `npx skills add` | 90K+ | Install-count ranking |
| officialskills.sh | Official vendor index | Browse only | `npx skills add <original-repo>` | 581+ | Curated (official only) |
| agentskill.sh | Universal registry | `/learn` / CLI / browse | `/learn @owner/skill` / CLI | 158K+ | Two-layer scan (0-100) |
| agent-skill.co | Curated index | Browse only | `git clone` | ~1K | None |

### 1. skills.sh (Vercel)

**Search:**
```bash
npx skills find <query>
```
Browse skills visually at https://skills.sh/

**Install:**
```bash
# Install a skill from a repo
npx skills add <owner/repo>

# Install a specific skill from a repo with multiple skills
npx skills add <owner/repo> --skill <name>
```

### 2. officialskills.sh (Browse-Only Index)

**What it is:** A browsable directory of OFFICIAL vendor-published skills. It is NOT a registry — it's a curated index that links to each skill's ORIGINAL GitHub source repo (from the actual vendor, e.g., anthropics/skills, microsoft/playwright-cli).

**Search:**
Browse https://officialskills.sh/
Filter by publisher (Anthropic, OpenAI, Microsoft, Google, etc.) or category.

**Install:**
Each skill on officialskills.sh links to its original GitHub repo. Install from the ORIGINAL source using the standard skills.sh mechanism:
```bash
npx skills add <original-owner/original-repo>
npx skills add <original-owner/original-repo> --skill <name>  # if multi-skill repo
```

**Context:**
- Backed by [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) — this is the INDEX/CATALOG, NOT the install source
- 48 development teams contributing
- 581+ skills as of April 2026
- Examples: anthropics/skills, openai/skills, sveltejs/ai-tools

### 3. agentskill.sh (158K+ Skills)

**What it is:** A large skill registry with built-in two-layer security scanning (server-side + client-side). Uses its own CLI (`ags`) and the `/learn` skill command.

**Search:**
```bash
# Browse
https://agentskill.sh/

# Via CLI (no /learn needed)
npx @agentskill.sh/cli search <query>
```

**Install Workflow:**

**Step 1 — Setup the `/learn` command (one-time):**
```bash
npx @agentskill.sh/cli@latest setup
```
This installs the `/learn` skill so you can search/install from within your agent.

**Step 2 — Install a specific skill:**
```bash
# Inside your agent chat:
/learn @owner/skill-name

# Or directly via CLI (no /learn needed):
npx @agentskill.sh/cli install <slug>
```

**Management commands (after /learn is installed):**
```
/learn list       # List installed skills
/learn update     # Check for updates
/learn remove     # Remove a skill
```

**Security:** agentskill.sh runs server-side static analysis on every skill across 12 threat categories, scoring 0-100. Skills below 30 require explicit confirmation. `/learn` runs a second client-side scan before writing files.

**API (programmatic):**
```
GET https://agentskill.sh/api/agent/skills/{owner}%2F{skill}/install
```
Returns JSON with skillMd, skillFiles, installPath.

### 4. agent-skill.co (Curated Index)

**What it is:** A small curated directory of hand-picked agent skills. Not a registry — just a browseable index linking to each skill's GitHub source.

**Search:**
Browse https://agent-skill.co/

**Install:**
Each skill links to its GitHub repo. Install via git clone or standard skills.sh if the repo follows SKILL.md format.

**Details:**
- Curated by Hailey Cheng
- Backed by [heilcheng/awesome-agent-skills](https://github.com/heilcheng/awesome-agent-skills)

## 4 Install/Delete Targets

| # | Target | Path |
|---|--------|------|
| 1 | Local project | `./.agents/skills/` (cwd) |
| 2 | OpenCode profile | `~/.config/opencode/profiles/<active-profile>/skills/` (auto-detect) |
| 3 | Global opencode | `~/.config/opencode/skills/` |
| 4 | Global agents | `~/.agents/skills/` |

### Important: Use `--skill <name>` to avoid extra skills

Many repos contain multiple skills. Without `--skill`, `npx skills add` installs ALL skills from the repo.
**Always use `--skill <name>`** to install only the specific skill the user asked for:
```bash
npx skills add <owner/repo> --skill <name>
```

### Install via `npx skills`

- **No flag** → project-local install to `./.agents/skills/<skill-name>/`
  ```bash
  npx skills add <source> --skill <name>
  ```

- **`-g` flag** → global install to `~/.agents/skills/<skill-name>/` (NOT opencode-specific)
  ```bash
  npx skills add <source> -g -y --skill <name>
  ```

- **Profile install** → install to global first, then **MOVE** (not copy) to profile, cleaning up global:
  ```bash
  # Step 1: Install globally (temporary)
  npx skills add <source> -g -y --skill <name>

  # Step 2: MOVE to profile path (use --agent opencode for correct routing)
  mkdir -p ~/.config/opencode/profiles/<active-profile>/skills/<name>/
  mv ~/.agents/skills/<name>/* ~/.config/opencode/profiles/<active-profile>/skills/<name>/
  rm -rf ~/.agents/skills/<name>/        # clean up global location

  # Step 3: Verify global is clean
  ls ~/.agents/skills/<name>/ 2>&1 || echo "Clean: skill removed from global"
  ```

  **Result:** Skill exists ONLY in the profile, nowhere else.

## Profile Auto-Detection

When the user wants to install to their OpenCode profile, determine the active profile in this order:

1. **Check `$OPENCODE_PROFILE`** environment variable — if set, use it directly
2. **Check `~/.config/opencode/opencode.jsonc`** — look for a `profile` field
3. **List profiles** — check `~/.config/opencode/profiles/*/`:
   - If only one profile exists, use it
   - If multiple profiles exist, ask the user which one
4. **Fallback** — ask the user: "Which profile would you like to install to?"

## Security Vetting Protocol (BEFORE EVERY INSTALL)

**Run the skill-vetter protocol before installing any skill from any provider.**

Source: `UseAI-pro/openclaw-skills-security/skill-vetter`

### Step 1: Source Metadata Check

Gather and present to the user:
- **Origin** — Which provider? (GitHub / skills.sh / officialskills.sh / agentskill.sh / agent-skill.co)
- **Author/Publisher** — Who created it? What is their reputation?
- **Stars** — GitHub stars or platform rating
- **Downloads** — Download count or installs
- **Last Update** — When was it last maintained?
- **Purpose** — A clear, concise statement of what the skill claims to do

### Step 2: Full File Review

Check **every file** in the skill for these 15 red flags:

1. `curl` / `wget` to unknown URLs — potential data exfiltration
2. Sending data to external servers — privacy leak
3. Requesting tokens / API keys / credentials — credential theft
4. Reading `~/.ssh`, `~/.aws`, `~/.config` — sensitive directory access
5. Reading OpenClaw private files: `MEMORY.md`, `USER.md`, `SOUL.md`, `IDENTITY.md`, `TOOLS.md`, `openclaw.config.json`
6. `base64` decode of opaque content — obfuscation
7. `eval()` / `exec()` with external input — code injection
8. Modifying files outside workspace — system tampering
9. Installing undeclared dependencies — supply chain risk
10. IP address connections (hardcoded IPs) — DNS evasion
11. Minified / obfuscated code blocks — hidden behavior
12. `sudo` / elevated permissions — privilege escalation
13. Accessing browser cookies / sessions — session hijacking
14. Writing to startup / init directories — persistence mechanisms
15. Network sockets / listeners — backdoor potential

### Step 2.5: Skill Dependency Check

Before installing, check if the skill depends on other skills:

1. **Check SKILL.md frontmatter** — Look for `requires`, `depends_on`, or `dependencies` fields
2. **Check SKILL.md body** — Search for references to other skills by name (e.g., "requires X skill", "needs X installed")
3. **Check for companion skills** — Run `npx skills add <owner/repo> --list` to see all skills in the same repo. Ask: "This repo also contains: [list]. Do you want any of these as well?"
4. **Check for MCP/tool dependencies** — Look for references to MCP servers, external services, or CLI tools (e.g., "requires shadcn/ui MCP")
5. **Check README** — If the repo has a README, check it for dependency documentation

Report findings to the user:

```markdown
**Skill Dependencies:**
- [skill-name] — requires [dep-1], [dep-2]
- Companion skills available in same repo: [list]
- MCP/tool requirements: [list]
- None found — skill is self-contained ✅
```

### Step 3: Permission Scope Assessment

Determine what the skill needs access to:
- **Reads** — what files or data does it read?
- **Writes** — what files or directories does it modify?
- **Executes** — what commands or scripts does it run?
- **Network** — does it make external connections? Are they necessary?

### Step 4: Risk Grade

Assign a risk grade based on findings:

| Grade | Criteria |
|-------|----------|
| **LOW** | No red flags, well-known author, clear purpose, minimal permissions |
| **MEDIUM** | Minor flags present (e.g., network needed for legitimate purpose, reads some config files) |
| **HIGH** | Several red flags present, obfuscated code, unclear author, broad permissions |
| **EXTREME** | Credential theft patterns, obfuscation / base64, exfiltration to unknown hosts, privilege escalation |

### Step 5: User Confirmation

Present the full vetting report to the user and explicitly ask **whether to proceed with installation**.

```markdown
## Skill Vetting Report: <skill-name>

**Provider:** ...
**Author:** ...
**Risk Grade:** [LOW | MEDIUM | HIGH | EXTREME]

**Findings:**
- Flag 1: ...
- Flag 2: ...
...

**Permissions Required:**
- Reads: ...
- Writes: ...
- Executes: ...
- Network: ...

**Do you want to proceed with installation? (yes/no)**
```

Wait for user confirmation before installing.

## Delete Workflow

When the user asks to remove or delete a skill:

1. **Identify the target** — ask or infer which target the skill is installed in:
   - Local project (`./.agents/skills/`)
   - OpenCode profile (`~/.config/opencode/profiles/<active-profile>/skills/`)
   - Global opencode (`~/.config/opencode/skills/`)
   - Global agents (`~/.agents/skills/`)

2. **Remove the skill directory** from the identified target path.

3. **Confirm success** — report what was removed.

### Target Paths for Deletion

| Target | Path |
|--------|------|
| Local project | `./.agents/skills/<skill-name>/` |
| OpenCode profile | `~/.config/opencode/profiles/<active-profile>/skills/<skill-name>/` |
| Global opencode | `~/.config/opencode/skills/<skill-name>/` |
| Global agents | `~/.agents/skills/<skill-name>/` |

**Example:**
> User: "Remove X from my profile"
> Action: Delete `~/.config/opencode/profiles/<active-profile>/skills/X/`
> Response: "Skill X has been removed from your profile."

## Update Workflow

When the user asks to update or check for updates on installed skills:

### For skills.sh / officialskills.sh skills:
```bash
npx skills check    # Check for updates
npx skills update   # Update all installed skills
```

### For agentskill.sh skills:
```
/learn update
```

### For manually installed skills (git clone):
```bash
cd <skill-directory>
git pull
```

### Note on limitations:
- `npx skills update` may re-add skills from multi-skill repos (known bug in v1.5.x)
- For profile-installed skills, re-copy from global dir after update
