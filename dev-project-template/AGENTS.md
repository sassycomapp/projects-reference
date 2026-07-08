# AGENTS.md — dev-project-template

> Inherits Global `AGENTS.md` in full. This file adds project-specific rules only.
> **Template:** Replace `[PLACEHOLDER]` markers when starting a new project.

## Project purpose

Project template for bootstrapping new Anvil.works projects. Copy to `C:\projects\dev-<name>/` and customise. Maintained in `C:\projects\dev-project-template/`.

---

## Definitions

- **project-library** — this repo. **[CODE-REPO-NAME]** — the separate live application code repo.

---

## Precedence order

1. Explicit user instruction → 2. Global `AGENTS.md` → 3. Hard Rules here → 4. ADRs → 5. Docs → 6. Skill defaults. Conflict = Default-to-Ask.

---

## Hard rules

### 1. anvil.yaml protection (enforced by opencode.json)
Never write to `anvil.yaml`.

### 2. Two-repo separation
- `project-library/` = planning, docs, ADRs, wireframes, pdlf artifacts. Not for Anvil.
- `[CODE-REPO-NAME]` = live Anvil code + `anvil.yaml`.
- Write to code repo only during coding phase, only code files, never `anvil.yaml`.

### 3. GBrain cross-contamination discipline
Tag every GBrain result with its source project. Cross-project decisions require explicit human confirmation. Scope queries with `--source [PROJECT-SLUG]`.

### 4. No time-based references
Use Phase > Stage > Task > Sub-task hierarchy.

### 5. No hardcoded colors
Every color through CSS custom properties.

### 6. Plan reads, Build writes (enforced by opencode.json)
Plan never writes. Build writes only with user approval.

### 7. Skill ownership
Ask which skill before starting any task.

### 8. No deviation
Execute as specified. Propose deviations first.

### 9. Binding documents
ADRs, specs, rule files are binding. Check before starting.

### 10. Mandatory project context reading
Read `README.md` before every task.

---

## Workspace structure

| Folder | Purpose |
|---|---|
| `pre-project/` | First-thoughts, scoping ideas |
| `pdlf/` | pdlf orchestrator artifacts |
| `adr/` | Architecture Decision Records |
| `docs/` | Durable documentation and specs |
| `wireframes/` | HTML wireframes |
| `screens/` | Screen HTML mockups |
| `wip/` | Active work-in-progress |
| `gstack-outputs/` | GStack-produced reports |
| `opencode-outputs/` | OpenCode-produced reports |
| `matt-skills-output/` | Matt Pocock skill outputs |

---

## Artifact governance

Every artifact gets saved. Build actions only. Confirm save location.

| Producer | Destination |
|---|---|
| GStack | `gstack-outputs/` |
| OpenCode | `opencode-outputs/` |
| Matt Pocock skills | `matt-skills-output/` |

File type overrides: docs → `docs/`, ADRs → `adr/`, wireframes → `wireframes/`, screens → `screens/`, pdlf → `pdlf/`.

---

## pdlf workflow

Managed using pdlf. Type `/pdlf` to init or resume.
- **pdlf home:** `C:\pdlf\`
- **State file:** `~/.gstack/projects/[PROJECT-SLUG]/pdlf-state.md`
- **GBrain source:** `[PROJECT-SLUG]` (isolated)

---

## Source of truth

- `project-library/adr/` — architectural decisions
- `project-library/docs/` — documentation and specs
- `[CODE-REPO-NAME]` — live code (read-only context)

---

## Access model

- Plan: reads both repos. Never writes.
- Build: writes in `project-library` with approval. Writes in code repo only during coding phase with case-by-case permission.

---

## Task completion reporting

- Reports → producer's output folder.
- `devlog-index.md` — report index.
- `judgement-trail.md` — decisions per task.
- Scaffold exception: create headers if missing.
