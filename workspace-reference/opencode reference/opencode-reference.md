# OpenCode Reference

## What is OpenCode

OpenCode is the AI coding agent that runs your workspace. It's the interface where you type prompts, run slash commands, and interact with the agent.

**Interface:** Web UI only (`opencode web --hostname 0.0.0.0` from WSL) | **Config:** `~/.config/opencode/` (global) + per-project `opencode.json`

---

## Two Modes

| Mode | What the agent can do | Default in |
|---|---|---|
| **Plan** | Read files, suggest changes, no edits | project-library, mb4ecom, mb5pdlf, dev-pdlf |
| **Build** | Read and edit files, run commands (with approval) | mb-3-cs code repo |

Switch with **Tab** key. Current mode shown in bottom-right corner.

---

## Day-to-Day Usage

### Start OpenCode

```bash
# From WSL, in the project directory you want to work in
cd /mnt/c/dev/dev-mb-3-cs/project-library
opencode web --hostname 0.0.0.0
```

Then open browser to `http://localhost:4096`.

### Ask the Agent a Question

Type naturally in the chat box:

> "What did we decide about onboarding?"
> "Search GBrain for login authentication"
> "Where is the LoginForm code?"

The agent uses GBrain, Grep, or file reading as needed.

### Run a Slash Command

Type `/` followed by the command name. See the skill reference docs for the full catalog.

### Start / Switch Sessions

| Shortcut | Action |
|---|---|
| **Ctrl+X, N** | New session |
| **Ctrl+X, L** | Session list |
| **Ctrl+X, R** | Rename session |
| **Ctrl+X, D** | Delete session |

### Session Navigation

| Shortcut | Action |
|---|---|
| **Up Arrow** | Parent session |
| **Down Arrow** | First child session |
| **Left/Right Arrow** | Previous/Next sibling session |

### Other Commands

| Shortcut / Command | Action |
|---|---|
| **Tab** | Switch Plan/Build mode |
| **Ctrl+P** | Command palette |
| **Ctrl+G** | Interrupt/stop agent mid-response |
| **Enter** | Submit prompt |
| **Shift+Enter** | New line (without sending) |
| **Esc** | Cancel current action |
| `/undo` | Undo agent's last change |
| `/redo` | Redo undone change |
| `/context-save` | Save working context |
| `/context-restore` | Restore previous context |
| `/share` | Share conversation via link |
| `/help` | Show help |

---

## Configuration Architecture

### Config Precedence (highest to lowest)

1. Per-project `opencode.json` (project root)
2. Global `~/.config/opencode/opencode.jsonc`
3. Remote/org config (if any)

Strictest rule wins on conflict. Model selection comes from the UI selector, not config files.

### Config Files

| File | Path | Purpose |
|---|---|---|
| Global config | `~/.config/opencode/opencode.jsonc` | Provider, permissions, MCP, instructions, agent restrictions |
| Global AGENTS.md | `~/.config/opencode/AGENTS.md` | Global behavior rules (132 lines) |
| Project config | `<project>/opencode.json` | Per-project permissions, default agent, instructions |
| Project AGENTS.md | `<project>/AGENTS.md` | Project-specific rules (under 200 lines each) |

---

## Global Config

**Path:** `~/.config/opencode/opencode.jsonc`

Key settings:
- `default_agent`: `"plan"`
- `instructions`: loads 4 global INDEX.md files (ADR, specs, policy, SOPs)
- `permission.edit`: `"ask"` with `**/anvil.yaml` denied
- `permission.bash`: `"ask"` with destructive commands denied (`rm -rf`, `git reset --hard`, `git push --force`)
- `agent.plan.permission.edit`: `"deny"` (Plan cannot write — deterministic)
- `external_directory`: allows `C:\mybizz\**`, `C:\projects\**`, `C:\dev\**`, `C:\pdlf\**`
- `mcp.gbrain`: local GBrain MCP server via bun

---

## Per-Project Configs

All project configs share a common pattern:

| Setting | Value |
|---|---|
| `default_agent` | `"plan"` (project-library) or `"build"` (code repos) |
| `instructions` | `["README.md"]` — loads README into context |
| `permission.edit` | `"ask"` with `**/anvil.yaml` denied |
| `permission.bash` | `"ask"` with destructive commands denied |
| `agent.plan.permission.edit` | `"deny"` (Plan cannot write) |
| `external_directory` | Allows read access to sibling repo |

### mb-3-cs code repo has extra protections:

| Protected file | Permission |
|---|---|
| `anvil.yaml` | `deny` |
| `parameters.yaml` | `deny` |
| `templates.yaml` | `deny` |
| `**/form_template.yaml` | `deny` |
| `standard-page.html` | `ask` |
| `theme.css` | `ask` |

---

## Project Config Files

| Project | Project-library config | Code repo config |
|---|---|---|
| mb-3-cs | `C:\dev\dev-mb-3-cs\project-library\opencode.json` | `C:\dev\dev-mb-3-cs\mb-3-cs\opencode.json` |
| mb4ecom | `C:\dev\dev-mb4ecom\mb4ecom-project-library\opencode.json` | `C:\dev\dev-mb4ecom\mb4ecom\opencode.json` |
| mb5pdlf | `C:\dev\dev-mb5pdlf\mb5pdlf-project-library\opencode.json` | `C:\dev\dev-mb5pdlf\mb5pdlf\opencode.json` |
| dev-pdlf | `C:\dev\dev-pdlf\opencode.json` | — |
| projects (root) | `C:\projects\opencode.json` | — |

---

## AGENTS.md Structure

All AGENTS.md files (global and project-level) follow the same pattern:

1. **Inheritance statement** — inherits Global AGENTS.md in full
2. **Project purpose** — what this workspace is for
3. **Definitions** — OpenCode, Skill, Task, Gate, Artifact
4. **Precedence order** — user instruction → global rules → hard rules → ADRs → docs → skill defaults
5. **Hard rules** — anvil.yaml protection, Plan/Build boundary, no time references, no hardcoded colors, etc.
6. **Mandatory project context reading** — README.md must be read before every task
7. **Workspace structure** — folder table
8. **Artifact governance** — routing by producer and file type
9. **Source of truth** — authoritative sources, fact rules
10. **Access model** — Plan reads, Build writes
11. **Task completion reporting** — reports, devlog, judgement trail

---

## Skills

**GStack skills:** `~/.config/opencode/skills/` (61 skills)
**Matt Pocock skills:** `~/.agents/skills/` (18 skills)

See `gstack-skills-reference.md` and `matt-skills-reference.md` for full catalogs.

---

## File Locations

| Item | WSL Path | Windows Path |
|---|---|---|
| Global config | `~/.config/opencode/opencode.jsonc` | `\\wsl.localhost\Ubuntu\home\dev-p\.config\opencode\` |
| Global AGENTS.md | `~/.config/opencode/AGENTS.md` | Same |
| Global skills | `~/.config/opencode/skills/` | Same |
| GStack skills source | `/mnt/c/mybizz/skills/` | `C:\mybizz\skills\` |
| Matt Pocock skills | `~/.agents/skills/` | `\\wsl.localhost\Ubuntu\home\dev-p\.agents\skills\` |

---

## MCP Integration

GBrain is registered as an MCP server in the global config. The agent calls GBrain tools automatically:

- `gbrain_search` — keyword search
- `gbrain_query` — hybrid vector + keyword search
- `gbrain_code_def` — find code symbol definitions
- `gbrain_code_refs` — find code symbol references

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Agent won't edit a file | Check if protected in `opencode.json` (`**/anvil.yaml` = deny) |
| Agent is in wrong mode | Press **Tab** to switch Plan/Build |
| MCP tools not available | Restart: `pkill -f "opencode web" && opencode web --hostname 0.0.0.0` |
| Wrong model | Check UI model selector — config file `model` is fallback only |
| Commands not showing | Check `.opencode/commands/` exists in project directory |
