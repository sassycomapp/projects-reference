# Workspace Docmap

Path register for all config, orchestration, and memory artifacts. Updated 2026-07-08.

---

## 1. Global config & rules

| File | Path (WSL) | Purpose |
|---|---|---|
| Global config | `~/.config/opencode/opencode.jsonc` | Provider, permissions, MCP, instructions, agent restrictions |
| Global AGENTS.md | `~/.config/opencode/AGENTS.md` | Global behavior rules (132 lines) |
| Global skills | `~/.config/opencode/skills/` | 61 GStack skills + 6 OpenClaw + 1 browser |
| Matt Pocock skills | `~/.agents/skills/` | 18 engineering skills |
| GBrain config | `~/.gbrain/config.json` | PGLite engine, embedding model, schema pack |
| GBrain database | `~/.gbrain/brain.pglite/` | PGLite database (323 MB) |
| GStack config | `~/.gstack/config.yaml` | Artifacts sync, telemetry, checkpoint mode |
| GStack home | `~/.gstack/` | Sessions, analytics, projects, learnings |

---

## 2. Project workspaces (`C:\dev\`)

### mb-3-cs

| Repo | Path | Default agent |
|---|---|---|
| project-library | `C:\dev\dev-mb-3-cs\project-library` | plan |
| code | `C:\dev\dev-mb-3-cs\mb-3-cs` | build |

### mb4ecom

| Repo | Path | Default agent |
|---|---|---|
| project-library | `C:\dev\dev-mb4ecom\mb4ecom-project-library` | plan |
| code | `C:\dev\dev-mb4ecom\mb4ecom` | build |

### mb5pdlf

| Repo | Path | Default agent |
|---|---|---|
| project-library | `C:\dev\dev-mb5pdlf\mb5pdlf-project-library` | plan |
| code | `C:\dev\dev-mb5pdlf\mb5pdlf` | build |

### dev-pdlf

| Repo | Path | Default agent |
|---|---|---|
| workspace | `C:\dev\dev-pdlf` | plan |

### dev-project-template

| Repo | Path | Default agent |
|---|---|---|
| template | `C:\dev\dev-project-template` | plan |

### dev-custom-dream-cycle

| Repo | Path |
|---|---|
| workspace | `C:\dev\dev-custom-dream-cycle` |

---

## 3. Shared resources (`C:\projects\`)

| Path | Purpose |
|---|---|
| `C:\projects\project-library-global\adr-global\` | Global ADRs (40 files, has INDEX.md) |
| `C:\projects\project-library-global\specifications-global\` | Global specifications (37 files, has INDEX.md) |
| `C:\projects\project-library-global\policy-global\` | Global policies (work in progress, has INDEX.md) |
| `C:\projects\project-library-global\standard-operating-procedures-global\` | Global SOPs (work in progress, has INDEX.md) |
| `C:\projects\project-library-global\docs-standard-global\` | Standard doc templates |
| `C:\projects\project-library-global\checklists-global\` | Global checklists |
| `C:\projects\project-library-global\templates-global\` | Global templates |

---

## 4. Shared infrastructure (`C:\mybizz\`)

| Path | Purpose |
|---|---|
| `C:\mybizz\gstack\` | GStack repo — binaries, skills, node_modules |
| `C:\mybizz\gbrain\` | GBrain application source (Node.js/Bun) |
| `C:\mybizz\skills\` | Skill definitions (~70 entries, 61 symlinked to OpenCode) |
| `C:\mybizz\matt-skills-teach\` | Shared teaching workspace for `/teach` skill |

---

## 5. Reference documents

| Document | Path | Lines |
|---|---|---|
| GBrain reference | `workspace-reference/gbrain-reference/gbrain-reference.md` | 263 |
| GStack reference | `workspace-reference/gstack-reference/gstack-reference.md` | 181 |
| GStack skills reference | `workspace-reference/gstack-reference/gstack-skills-reference.md` | 168 |
| Matt-skills reference | `workspace-reference/matt-skills reference/matt-skills-reference.md` | 376 |
| OpenCode reference | `workspace-reference/opencode reference/opencode-reference.md` | 216 |
| Workspace docmap | `workspace-reference/system config reference/workspace-docmap.md` | This file |

---

## 6. Per-project artifacts

Each project has these standard directories (created as needed):

| Directory | Purpose | Created by |
|---|---|---|
| `adr/` | Architecture Decision Records | First ADR written |
| `.scratch/` | Local markdown issue tracker | `/setup-matt-pocock-skills` |
| `.out-of-scope/` | Rejected issues | `/setup-matt-pocock-skills` |
| `matt-skills-output/` | Matt Pocock skill outputs | `/setup-matt-pocock-skills` |
| `gstack-outputs/` | GStack skill outputs | First gstack skill used |
| `docs/agents/` | Agent config (3 files) | `/setup-matt-pocock-skills` |
| `CONTEXT.md` | Domain glossary | `/domain-modeling` or `/grill-with-docs` (lazy) |

---

## 7. GStack SLUG paths

Each project gets a GStack internal directory at `~/.gstack/projects/{SLUG}/` with 8 subdirectories:

| SLUG | Project |
|---|---|
| `sassycomapp-projects-mgt` | C:\projects root |
| `sassycomapp-project-library` | mb-3-cs project-library |
| `sassycomapp-project-library-mb4ecom` | mb4ecom project-library |
| `sassycomapp-project-library-mb5pdlf` | mb5pdlf project-library |
| `sassycomapp-dev-pdlf` | dev-pdlf |

Subdirectories: `ceo-plans/`, `designs/`, `checkpoints/`, `retros/`, `specs/`, `security-audits/`, `health/`, `qa-reports/`

---

## 8. PDLF framework (`C:\pdlf\`)

| Path | Purpose |
|---|---|
| `C:\pdlf\skill\` | PDLF skill definitions |
| `C:\pdlf\templates\` | Pre-project document templates |
| `C:\pdlf\registry\` | Project slug resolution |
| `C:\pdlf\docs\` | PDLF documentation |

---

## 9. Environment

- **Interface:** OpenCode Web UI only (`opencode web` from WSL)
- **Shell:** `/bin/bash` in WSL (Ubuntu 22.04.5 LTS)
- **Model:** Selected from UI picker, not hardcoded
- **GBrain engine:** PGLite (migrated from Supabase), v0.42.57.0
- **GBrain schema pack:** `gbrain-everything` (code-aware)
- **GStack version:** 1.58.1.0
