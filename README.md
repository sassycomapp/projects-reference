# C:\projects — Workspace Overview

## What this workspace is

All projects in this workspace are Anvil.works projects. They all use the Anvil Material 3 theme, the M3 dependency, and the MUI overlay. Many aspects common to all projects are saved in `C:\dev\project-library-global\` and referenced from all projects.

All project repos, including `C:\projects`, are registered as sources on GBrain and configured with OpenCode, GStack, GitHub, and Matt Pocock skills.

---

## Two-repo architecture

Every Anvil-based project has two repos:

| Repo | Purpose |
|---|---|
| **Code repo** | Live Anvil code + `anvil.yaml`. Goes to Anvil via GitHub VC. |
| **Documentation repo** (`project-library/`) | Planning, docs, ADRs, wireframes, specifications. Does NOT go to Anvil. |

Project folders only hold documentation which is durable and canonical to that specific project. Everything general or global lives in `C:\projects`.

---

## Project file prefixes

| Prefix | File type |
|---|---|
| `spec-` | Specifications |
| `policy-` | Policy documents |
| `ADR-` | Architecture Decision Records |
| (No prefix) | Canonical durable project documents |
| `screen-` | Screens (HTML docs) |
| `wireframe-` | Wireframes (HTML docs) |
| `tmp-` | Template documents |
| `chk-` | Checklist documents |
| `sop-` | Standard Operating Procedures |
| `custom-` | Custom skills |

---

## Shared resources: project-library-global

`C:\projects\project-library-global` holds folders containing files shared across all projects. These folders have the suffix `-global`. They are maintained in a central repository for ease of maintenance and updating, so that different copies of the same files do not accumulate across projects.

| Folder | Contents |
|---|---|
| `project-library-global/adr-global/` | Global Architecture Decision Records |
| `project-library-global/checklists-global/` | Global checklists |
| `project-library-global/docs-standard-global/` | Standard SaaS/dev document templates (reference, not production-ready) |
| `project-library-global/policy-global/` | Global policies |
| `project-library-global/specifications-global/` | Global specifications |
| `project-library-global/templates-global/` | Global templates |

### docs-standard-global

These are standard SaaS/dev documents. The format in these files is not necessarily intended to be used as-is in the projects. They indicate the documents that will be required in a project and provide an example of what should go into them — but not a definitive template.

### Reusable documents

No project will use all the global ADR documents, checklists, policies, specifications, SOPs, or templates. Individual projects reference the specific documents in those folders that pertain to them.

### State of global documents

All documents in `C:\projects\project-library-global` have been hurriedly and haphazardly assembled. Many have been copied in directly from other project folders. The documents need to be updated and in some cases created. Documents with the prefix `mt-` or empty documents require writing. Some folders only have a single document, which is really just a placeholder for the documents that must be created in that folder.

---

## Project template

`C:\dev\project-template\` is currently under development. It is intended to kick-start a new project — the developer copies the project-template into the new project and that folder becomes the project library for the new project.

When the dev-project-template project is complete, it will move into `C:\projects\project-library-global`.

---

## Standard Operating Procedures

These are predetermined Modus Operandi (M.O.'s). Each file constitutes the standard method of executing a specific task — one task per file. Each SOP file has the prefix `sop-`.

---

## Project lifecycle

When a dev project has been completed and deployed, the project is live and the dev project gets archived. It will no longer be directly in the `C:\projects` folder but will move to `C:\projects\deployed-projects`.

---

## PDLF

It is possible for two or more projects to be in a state of development in the PDLF at any one time. PDLF must manage this — do not get lines crossed between the projects.

---

## GBrain sync

Sync jobs can get stuck waiting — `gbrain serve` holds the PGLite lock. To resolve: stop `serve`, run sync/embed/extract via CLI, then restart `serve`.

See: `C:\projects-reference\workspace-reference\workflow reference\daily-ops.md` for help.
