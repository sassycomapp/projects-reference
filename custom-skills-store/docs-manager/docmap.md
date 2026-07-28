# Mybizz Division — Docmap

**Purpose:** Single canonical navigation map for the Mybizz division — where every folder lives, what it is for, and where new files should go. Agents read this to understand the territory.

**Date:** 2026-07-23
**Verified:** 2026-07-23

**Companion:** `project-inventory.md` (same folder) — project-level paths, GitHub repos, GBrain sources, GStack artifact paths.

---

## 1. Division Root — `C:\mybizz\`

```
C:\mybizz\
├── archive/                    ← Long-term reference store (not backup, not WIP)
│   ├── Anvil_Methods/          ← Original Anvil specifications
│   ├── Model Assessments/      ← Historical model confidence reports
│   ├── artifact-saving-issue/  ← Resolved GStack investigation (11 files + resolution report)
│   ├── ci-based-checks-format/ ← Archived check specs
│   ├── local-testing-example/  ← Archived testing example
│   ├── mybizz-core-methods/    ← Archived ADRs + methods
│   └── prompts-example/        ← Archived prompt examples
├── backup-mybizz/              ← BACKUP ONLY. Agents may read (with caution) but never edit or delete.
├── Desktop/                    ← Active working desktop environment
│   ├── Notebooks/              ← OneNote notebooks
│   ├── Notebooks-look for good/ ← Additional notebooks
│   ├── pc-mapping/             ← PC hierarchy diagrams (HTML)
│   └── wip/                    ← Personal daily scratchpad (contains todo.txt)
├── gbrain/                     ← Installed tool (not user-managed)
├── gstack/                     ← Installed tool (not user-managed)
├── logs/                       ← System and tool logs
│   ├── docs-manager/           ← docs-manager skill run logs
│   │   ├── Learnings/          ← docs-manager institutional memory, one file per learning
│   │   ├── in-progress/        ← live record of the currently-active run only
│   │   ├── last-completed-run/ ← single slot, full record of the most recent completed run
│   │   └── abandoned-runs/     ← discarded incomplete runs, never deleted
│   ├── github-logs/            ← Commit/push reports and closing git-status snapshots
│   └── gbrain-logs/            ← GBrain sync per-source logs
├── Mgt/                        ← Business documents ONLY (financial, planning, management)
│   ├── Davids Management .xlsx
│   ├── namecheap-order-196053207.pdf
│   └── obsolete/               ← Mgt-level obsolete items. Developer purges only.
├── scripts/                    ← Global utility scripts for this PC
└── skills/                     ← Installed tool (not user-managed)
```

---

## 2. Development Hub — `C:\dev\`

Where projects are developed. Active projects have the `dev-` prefix. Folders without `dev-`
prefix are support folders, including `dev-root/` which holds division-level inventory and
mapping documents.

```
C:\dev\
├── dev-mb-3-cs/                ← ACTIVE PROJECT (mb-3-cs)
│   ├── mb-3-cs/                ← Code repo
│   ├── mb-3-cs-project-library/        ← Docs repo
│   └── wip/                    ← Project WIP (contains todo.md)
├── dev-mb4ecom/                ← ACTIVE PROJECT (mb4ecom)
│   ├── mb4ecom/                ← Code repo
│   ├── mb4ecom-project-library/ ← Docs repo
│   └── wip/                    ← Project WIP (contains todo.md)
├── dev-mb5pdlf/                ← ACTIVE PROJECT (mb5pdlf)
│   ├── mb5pdlf/                ← Code repo
│   ├── mb5pdlf-project-library/ ← Docs repo
│   └── wip/                    ← Project WIP (contains todo.md)
├── dev-pdlf/                   ← ACTIVE PROJECT (PDLF — docs-only, no separate code repo)
│   ├── pdlf/                   ← Output staging for deployment
│   ├── wip/                    ← Project WIP (contains todo.md)
│   └── (many subfolders — see dev-pdlf/docs-local/docmap.md)
├── dev-root/                    ← Division-level inventory and mapping docs
│   ├── docmap.md                ← Full division hierarchy map
│   └── project-inventory.md     ← Project registry (paths, repos, GBrain, GStack)
├── obsolete/                   ← Dev-level obsolete. Developer purges only.
├── starting-prompt.txt          ← Session-template file for agent tasks
├── project-library-global/     ← Shared standards and reference for all projects
│   ├── adr-global/             ← Global architectural decision records
│   ├── checklists-global/      ← Global checklists
│   ├── docs-standard-global/   ← Standard doc templates
│   ├── guides/                 ← Global how-to guides
│   ├── policy-global/          ← Global policies
│   ├── specifications-global/  ← Global specifications
│   ├── standard-operating-procedures-global/ ← Global SOPs
│   └── templates-global/       ← Global templates
└── project-template/           ← Skeleton for new projects (import into new dev-* folder)
```

---

## 3. PDLF Framework — `C:\pdlf\` — **excluded from Docs Manager, see note**

**Current status: placeholder, not live.** This is the remains of a first, aborted attempt at
building PDLF — implemented halfway, then abandoned. It is not the deployed product yet; it may
have reference value in showing how that earlier attempt was structured, but nothing here
should be treated as current. dev-PDLF is where the real framework is being built; once that
concludes, the finished framework will be deployed here.

**Deliberately excluded from Docs Manager's walk root** (SKILL.md Section 1.1) — scanning a
placeholder that nobody is maintaining would just generate drift noise about content that isn't
meant to be current. **This should be revisited once dev-PDLF actually deploys here** — at that
point `C:\pdlf\` becomes live and belongs back in scope as a fourth walk root.

```
C:\pdlf\
├── .scratch/
├── docs/
├── registry/
└── skill/
```

---

## 4. Reference Library — `C:\projects-reference\`

Reference documents, custom skills, and completed projects. **Active git repo** —
`https://github.com/sassycomapp/projects-reference` (branch `main`). Note: `.git` was removed
on 2026-07-20 and this location was briefly out of scope for docs-manager; a new GitHub repo was
created afterward and it is now tracked normally again as an Active Repository — see
`project-inventory.md`.

```
C:\projects-reference\
├── custom-skills-store/        ← Custom skills (built or planned)
├── deployed-projects/          ← Completed project storage (currently empty)
└── workspace-reference/        ← How-to references for tools, apps, custom methods
    ├── Anvil-reference/
    ├── cloudflare-reference/
    ├── gbrain-reference/
    ├── gstack-reference/
    ├── hotkeys- reference/
    ├── markdown reference/
    ├── matt-skills reference/
    ├── model selection reference/
    ├── namecheap reference/
    ├── opencode reference/
    ├── system config reference/
    └── workflow reference/
        ├── daily-ops.md        ← Daily operations quick reference
        └── session-opening-prompt.md
```

---

## 5. Excluded from mapping

- `C:\_Data-not mybizz\` — personal interests, not Mybizz, not managed
- `C:\_ BIG BACKUP\` — read-only backup archive, never edited

---

## 6. File Placement Rules

### Where do new files go?

| File type | Destination |
|---|---|
| Project code | `C:\dev\dev-{project}\{slug}\` |
| Project docs | `C:\dev\dev-{project}\{slug}-project-library\` |
| Project WIP / todo | `C:\dev\dev-{project}\wip\todo.md` |
| Global standards (ADRs, policies, specs, SOPs) | `C:\dev\project-library-global\{category}\` |
| Global how-to guides | `C:\dev\project-library-global\guides\` |
| Business documents (financial, planning) | `C:\mybizz\Mgt\` |
| Daily working files / scratchpad | `C:\mybizz\Desktop\` or `C:\mybizz\Desktop\wip\` |
| Completed work for long-term reference | `C:\mybizz\archive\` |
| Temporary trash during a session | `C:\dev\obsolete\` (dev items) or `C:\mybizz\Mgt\obsolete\` (mgt items) |
| Custom skills | `C:\projects-reference\custom-skills-store\` |
| Reference docs (how-to, tool docs) | `C:\projects-reference\workspace-reference\` |
| Completed projects | `C:\projects-reference\deployed-projects\` |
| Backups | `C:\mybizz\backup-mybizz\` (new folder per backup exercise) |
| System and tool logs | `C:\mybizz\logs\{tool}\` (e.g., `C:\mybizz\logs\gbrain-logs\`, `C:\mybizz\logs\github-logs\`) |

### archive vs obsolete

| | archive | obsolete |
|---|---|---|
| **Purpose** | Long-term reference store | Temporary trash during a session |
| **Permanence** | Permanent | Purged by developer when session ends and work is stable |
| **Who decides** | Developer | Developer |
| **Location** | `C:\mybizz\archive\` | `C:\dev\obsolete\` or `C:\mybizz\Mgt\obsolete\` |

### Per-project WIP convention

Every active project has `C:\dev\dev-{project}\wip\todo.md`. The todo.md is project-specific — no central todo across all projects. WIP folders are outside both the code repo and the docs repo.

---

## 7. Tool Installations (not user-managed)

| Tool | Location | Purpose |
|---|---|---|
| GStack | `C:\mybizz\gstack\` | Skill framework, binaries, browse daemon |
| GBrain | `C:\mybizz\gbrain\` | Knowledge base, search, embeddings |
| Skills | `C:\mybizz\skills\` | Skill definitions (~70 entries) |
| GStack runtime (OpenCode) | `~/.config/opencode/skills/gstack/` | Runtime root with bin/, browse/dist/, design/dist/ |

---

## 8. Key Companion Documents

| Document | Location | Purpose |
|---|---|---|
| `docmap.md` | `C:\dev\dev-root\docmap.md` | THIS FILE — full division hierarchy map |
| `project-inventory.md` | `C:\dev\dev-root\project-inventory.md` | Project registry (paths, repos, GBrain, GStack) — single source of truth for repo/remote tracking |
| `README.md` | `C:\mybizz\README.md` | Division workspace overview |
| Global AGENTS.md | `~/.config/opencode/AGENTS.md` | Global agent behavior rules |
| Project AGENTS.md | `C:\dev\dev-{project}\AGENTS.md` | Per-project agent rules |
| daily-ops.md | `C:\projects-reference\workspace-reference\workflow reference\daily-ops.md` | Daily operations quick reference |
| dev-pdlf docmap | `C:\dev\dev-pdlf\docs-local\docmap.md` | PDLF project document map |
| docs-manager skill | `~/.config/opencode/skills/docs-manager/SKILL.md` | Skill definition — document inventory, update rules, workflow |
| | Windows: `\\wsl.localhost\Ubuntu\home\dev-p\.config\opencode\skills\docs-manager\SKILL.md` | |
| docs-manager sub-agents (5 files) | `~/.config/opencode/agents/docs-manager-backup.md`, `docs-manager-scan.md`, `docs-manager-apply.md`, `docs-manager-filesystem.md`, `docs-manager-commit.md` | One sub-agent per phase (0 backup, 1 scan, 4a text edits, 4b file-system operations, 6 commit/push), each with only the permissions its phase needs |
| | Windows: `\\wsl.localhost\Ubuntu\home\dev-p\.config\opencode\agents\` | |

**Retired:** `scaffold-system.html` is no longer a managed companion document — it was a
one-time HTML visual aid, now archived to `C:\mybizz\archive\` as a historical record rather
than actively maintained.
