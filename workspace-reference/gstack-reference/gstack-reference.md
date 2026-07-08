# GStack Reference

## What is GStack

GStack is the workflow layer. It provides slash commands (skills) for planning, building, reviewing, and shipping software. GBrain = memory (stores and searches). GStack = workflow (plans, builds, ships).

**Repo:** `/mnt/c/mybizz/gstack/` | **Bin:** `/mnt/c/mybizz/gstack/bin/` | **Skills:** `~/.config/opencode/skills/`

---

## Authority Model

1. User decision in the current session
2. Authoritative repo files and ADRs
3. OpenCode rules (global + project AGENTS.md, opencode.json)
4. GStack workflow conventions
5. GBrain advisory memory

When GStack workflow expectations conflict with OpenCode config, project rules, or user instruction — the stricter layer wins.

---

## Workflows

Three core workflow families:

### docs-plan (project-library)

**Use when:** analysis, validation, architecture, planning, ADR work, prompt authoring.
**Agent:** plan (primary)
**Skills:** `/office-hours`, `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review`

**Mandatory initial steps:**
1. Consult `wip/todo.md` — incorporate relevant items, mark completed as `[DONE]`
2. Consult ADR folder — treat as authoritative, create new ADRs for new decisions

**Exit condition:** planning objective complete, OR build-ready handoff prepared, OR canonical tracking updated.

### code-plan-build (code repos)

**Use when:** implementation, refactoring, migration, code validation, file edits.
**Agent:** build (primary), plan for complex/risky work
**Skills:** `/investigate`, `/review`, `/qa`, `/ship`

**Mandatory initial steps:**
1. Confirm repo path
2. Consult `wip/todo.md` for relevant items
3. Consult ADRs for architectural baseline

**Mandatory checks before edits:** repo path confirmed, protected files checked, backup rule applied, timestamp rule applied, write scope checked.

**Exit condition:** implementation complete, validation recorded, tracking requirements handed off.

### status-handover (cross-workspace)

**Use when:** session close, checkpoint, handover for continuation.
**Agent:** plan (primary)
**Skills:** `/context-save`, `/context-restore`

**Exit condition:** current state captured clearly enough for a fresh session to resume without reconstructing context.

---

## Handoff Contract

Work that changes ownership (Plan↔Build, docs↔code, session↔session) must produce a structured handoff.

### Plan→Build template

```
Task summary:
Scope in:
Scope out:
Files read:
Relevant ADRs:
Files expected to change:
Files that must not change:
Key constraints:
Implementation steps:
Validation required:
Tracking handoff required (devlog / todo / ADR):
```

### Build→Plan template

```
Completion status:
Files changed:
Backup confirmation:
Protected-file confirmation:
Validation result:
Implementation summary:
Proposed devlog handoff:
Proposed todo handoff:
Proposed ADR updates:
```

### Quality bar

A handoff is acceptable only if: short and specific, file paths explicit, scope boundaries clear, ADRs and tracking expectations visible, unresolved risks listed, sufficient for a new session to continue without reconstructing context.

---

## File Locations

| Item | Location |
|---|---|
| GStack repo | `/mnt/c/mybizz/gstack/` |
| GStack bin scripts | `/mnt/c/mybizz/gstack/bin/` |
| GStack skills (source) | `/mnt/c/mybizz/skills/` |
| GStack skills (symlinked) | `~/.config/opencode/skills/` |
| GStack home | `~/.gstack/` |
| GStack brain remote | `~/.gstack-brain-remote.txt` |
| GStack brain worktree | `~/.gstack-brain-worktree/` |
| GStack outputs | `gstack-outputs/` (per project) |

### Open GStack folders in Windows Explorer

```bash
# GStack home
explorer.exe "$(wslpath -w /home/dev-p/.gstack/)"

# GStack projects
explorer.exe "$(wslpath -w /home/dev-p/.gstack/projects/)"
```

Always use `wslpath -w` from WSL — using a Linux path directly opens the wrong location.

---

## Brain Sync

GStack brain sync pushes working state (learnings, plans, retros) to a private git repo.

| Command | Description |
|---|---|
| `bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --once` | Sync pending changes |
| `bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --status` | Check sync health |
| `bash /mnt/c/mybizz/gstack/bin/gstack-gbrain-detect` | GBrain + GStack status JSON |

---

## Artifact Locations

GStack skills save to `~/.gstack/projects/$SLUG/` internally. Per AGENTS.md, copies go to `gstack-outputs/`.

| Skill | Internal save | Copy to |
|---|---|---|
| `/office-hours` | `~/.gstack/projects/$SLUG/` | `gstack-outputs/` |
| `/plan-ceo-review` | `~/.gstack/projects/$SLUG/ceo-plans/` | `gstack-outputs/` |
| `/plan-eng-review` | `~/.gstack/projects/$SLUG/` | `gstack-outputs/` |
| `/design-consultation` | `~/.gstack/projects/$SLUG/designs/` | `gstack-outputs/` + `DESIGN.md` → `docs/` |
| `/context-save` | `~/.gstack/projects/$SLUG/.context/` | — |
| `/retro` | `~/.gstack/projects/$SLUG/retros/` | `gstack-outputs/` |

---

## Role Map

| Workspace | Default agent | Workflow family | Primary responsibility |
|---|---|---|---|
| project-library | plan | docs-plan | ADRs, specs, planning, devlog/todo |
| code repos | build | code-plan-build | Implementation, refactors, validation |
| cross-workspace | plan | status-handover | Session close, checkpoints, handoffs |

**You:** choose workspace, decide risk thresholds, accept/reject plans and handoffs.
**Agents:** propose plans and handoffs, keep devlog/todo/ADRs in sync, respect permissions.

---

## OpenCode Integration

GStack is exposed via skills (slash commands), not a separate CLI. The agent chooses which skill to invoke based on the task. No `.opencode/gstack` config exists — this is deliberate to keep authority with OpenCode.

| OpenCode Command | What it does |
|---|---|
| `/sync-gbrain` | Sync GBrain (incremental) |
| `/sync-gbrain --dream` | Build call graph |
| `/setup-gbrain` | One-time GBrain setup |

For the full skill catalog (59 skills), see `gstack-skills-reference.md`.
