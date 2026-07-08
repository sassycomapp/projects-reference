# AGENTS.md — projects

> Inherits Global `AGENTS.md` in full. Where this file is silent, global rules apply. Where this file adds detail, it may only tighten, never loosen.

## Project purpose

This is the umbrella workspace for all Anvil.works projects. It houses shared resources in `project-library-global/` (global ADRs, checklists, docs, policies, specifications, templates), individual project workspaces (e.g. `dev-mb-3-cs/`, `dev-mb4ecom/`, `dev-pdlf/`), and the project template (`dev-project-template/`). Each individual project has its own `AGENTS.md` and `README.md`. This root `AGENTS.md` governs only the `C:\projects` workspace itself.

---

## Definitions

- **OpenCode** — the host agent/orchestrator (Plan agent + Build agent).
- **Skill** — Gstack, Gbrain, Matt Pocock skills, Claude Wireframe, or a custom skill.
- **Task** — a discrete unit of work with a defined completion condition.
- **Gate** — a checkpoint requiring explicit user approval before proceeding.
- **Artifact** — any file produced during a session: reports, reviews, wireframes, screens, checklists, docs, ADRs.
- **project-library-global** — shared resource folder at `C:\projects\project-library-global/`.

---

## Precedence order (highest to lowest)

1. Explicit user instruction in the current session
2. Global `AGENTS.md` rules
3. Hard Rules in this file
4. ADRs (project-level `adr/` directories)
5. Specifications and durable docs (project-level `docs/`)
6. A skill's own default behavior

Conflict triggers Default-to-Ask.

---

## Hard rules

### 1. anvil.yaml protection (enforced by opencode.json)
Never write to `anvil.yaml`. Stop and ask if a task appears to require changing it.

### 2. Plan reads, Build writes (enforced by opencode.json)
Plan never writes. Build writes only where user has approved. No exceptions without explicit, case-by-case confirmation.

### 3. No writing to individual project workspaces
Do not write to project workspaces unless that project's `AGENTS.md` permits it and the user has approved the specific write.

### 4. Skill ownership
Ask the user which skill should be used before starting any task. Do not default to general-purpose execution.

### 5. No deviation from the given task
Execute the task as specified. If a deviation would be better, propose it and get approval first.

### 6. Binding documents
ADRs, specs, and rule files are binding constraints. Check for applicable documents before starting. Flag conflicts — do not silently override.

### 7. Mandatory project context reading
Read the project's `README.md` before every task alongside the `AGENTS.md` files.

---

## Workspace structure

| Folder | Purpose |
|---|---|
| `project-library-global/` | Shared resources: global ADRs, checklists, docs, policies, specifications, templates |
| `dev-*/` | Individual project workspaces |
| `dev-project-template/` | Template for bootstrapping new projects |

---

## Artifact governance

When working in a specific project, follow that project's artifact governance. When working at the `C:\projects` level, save to the relevant project's output folder.

Global file routing:

| File type | Destination |
|---|---|
| Global ADRs | `project-library-global/adr-global/` |
| Global checklists | `project-library-global/checklists-global/` |
| Global docs | `project-library-global/docs-standard-global/` |
| Global policies | `project-library-global/policy-global/` |
| Global specifications | `project-library-global/specifications-global/` |
| Global templates | `project-library-global/templates-global/` |

---

## GStack binary path override

Always use `/mnt/c/mybizz/gstack/bin/<binary-name>` directly. Do not attempt host-detection logic.

---

## Matt Pocock skills artifact governance

- `domain-modeling`, `grill-with-docs` → `CONTEXT.md` at project root. ADR output → `adr/`.
- `to-prd`, `to-issues`, `triage` → `.scratch/`. No redirect.
- `setup-matt-pocock-skills` → `docs/agents/` and `CLAUDE.md`. No redirect.
- `handoff`, `improve-codebase-architecture` → keep `%TEMP%` save + copy to `matt-skills-output/`.
- `teach` → redirect to `C:\mybizz\matt-skills-teach`.
- Any other Matt Pocock skill → ask the user.

---

## Source of truth

- `project-library-global/` — shared global resources
- Individual project `AGENTS.md` files — authoritative for that project
- `README.md` files at each level — project-specific context

Fact rule: do not use training data or memory for project-specific facts. GBrain research must be validated before use.

---

## Task completion reporting

When working in a project, follow that project's reporting mechanics. When at `C:\projects` level, reports go to the relevant project's `opencode-outputs/` folder.
