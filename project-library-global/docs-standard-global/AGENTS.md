# AGENTS.md — dev-pdlf

> This file inherits the Global `AGENTS.md` (dev-p) in full. Where this file is silent, the
> global rules apply. Where this file adds detail, it may only tighten global rules, never loosen
> them — this applies in particular to the Plan/Build write boundary below.

## Definitions

- **OpenCode** — the host agent/orchestrator operating in this workspace, encompassing the Plan agent and Build agent (see Global `AGENTS.md`).
- **Skill** — any of Gstack, Gbrain, Matt Pocock skills, Claude Wireframe, or a custom skill, once invoked to carry out a task.
- **Task** — a discrete unit of work with a defined completion condition, whether given directly by the user or delegated to a skill.
- **Gate** — a checkpoint in a workflow that requires explicit user approval before the next step may proceed.
- **Artifact** — any file produced during a session: reports, reviews, wireframes, screens, checklists, docs, ADRs, etc.

## Project purpose

This is the development workspace for the PDLF (Program Design Lifecycle Framework) — a
contiguous, stage-gated, memory-compounding software development strategy that operates
through OpenCode and GStack. The `dev-` prefix denotes that PDLF is currently under
development here. When dev-pdlf is concluded, it will be migrated to and replace `C:\pdlf\`
as the canonical system home.

Throughout this file, references to "a project," "the target project," or similar project-level
terms are generic — they describe the role or behavior expected of any PDLF-managed project, not
any one specific project.

Its purpose is to:
- develop and maintain the PDLF skill definition (`skill/SKILL.md` and variants),
- produce and maintain PDLF methodology documentation (`docs/`),
- record architectural and design decisions (`docs/adr/`),
- house PDLF pre-development research and reference materials (`docs/pre-pdlf/`),
- serve as the canonical home for the PDLF skill as consumed by OpenCode.

---

## Precedence order (highest to lowest)

1. Explicit user instruction given in the current session
2. Global `AGENTS.md` rules that this file may not loosen (e.g. Plan never writes)
3. Hard Rules in this file
4. ADRs (`docs/adr/`)
5. PDLF documentation (`docs/`)
6. A skill's own default behavior

A conflict at any level triggers Default-to-Ask (see Global `AGENTS.md`) — never a silent resolution in either direction.

---

## Hard rules (always apply)

### 1. anvil.yaml protection
Never write to `anvil.yaml`.
- Treat it as Anvil-managed metadata/configuration.
- Do not create, edit, reformat, regenerate, normalize, merge-edit, or patch it.
- Do not include it in bulk refactors or automated file updates.
- If a task appears to require changing it, stop and ask the user for explicit approval.
- Assume Anvil is the authoritative writer for `anvil.yaml`.

### 2. No hardcoded colors
No hardcoded colors permitted in the application.
- Every color must resolve through a CSS custom property from the palette system.
- No literal hex, `rgb()`, `rgba()`, or `hsl()` values — except MUI card-shadow `hsla()` values as defined in the MUI aesthetic overlay, and the status semantic exceptions: `#00b958` (success), `#ff9469` (warning), `#ff000c` (error).
- See `docs/DESIGN_palette_section.md` for authoritative token definitions.

### 3. No time-based references in planning docs
Never use time-based references (weeks, days, hours) in planning documents.
- Use the Phase > Stage > Task > Sub-task hierarchy instead.
- Write "Phase 2, Stage 2.1" or "Task T2.1-003" — not "2 weeks" or "3 days."
- Applies to all planning docs, build plans, roadmaps, and implementation schedules.

### 4. Documentation standards
- ADRs follow the standard `NNNN-title-slug.md` naming pattern.
- PDLF documentation in `docs/` must be maintained as accurate — stale docs are a failure condition.

### 5. Plan reads, Build writes — no exceptions, applies everywhere in this repo
This project follows the Global `AGENTS.md` write boundary at full strength, with no local loosening:
- The Plan agent never writes anywhere in this repo — not docs, not skill files, not reports. It reads and proposes only.
- The Build agent is the only agent that writes in this repo, and only where the user has approved the specific change.
- Any exception for Plan to write must be confirmed explicitly, case by case, for that one write — never as a standing permission, never as a per-task or per-project approval.

### 6. PDLF skill integrity
The canonical PDLF skill definition is `skill/SKILL.md` (or its active version as designated
by the user). Changes to the skill definition must:
- be explicitly approved by the user,
- maintain backward compatibility with existing PDLF-managed projects unless a migration
  path is explicitly defined,
- be documented in `docs/adr/` as an architectural decision.

### 7. No writing to PDLF-managed projects
- No agent or skill may write to any PDLF-managed project's code repo under any circumstance until that project has explicitly entered its coding phase.
- Even once coding has started, no agent may decide unilaterally to write to a PDLF-managed project. Every attempted write must first be posed to the developer for explicit, case-by-case permission — one-time blanket approval does not apply, and this cannot be satisfied by a standing project-level setting.
- This applies equally to Build acting directly and to any skill acting on Build's behalf. Plan never writes to a PDLF-managed project under any circumstance (see Rule 5).
- This workspace develops the PDLF framework itself. It does not write to projects managed by PDLF (e.g., mb-3-cs, mb4ecom, mb5pdlf) unless explicitly instructed for a specific framework update that targets those projects.

### 8. Skill ownership — no usurping
Inherits the Global "Skill / tool continuity" principle. Project-specific addition:
- The skills in scope for this project are Gstack, Gbrain, Matt Pocock skills, Claude Wireframe, and custom skills.
- Before starting any new task, OpenCode must stop and ask the user which skill (if any) should be used, or propose a skill and wait for confirmation, rather than defaulting to its own general-purpose execution.

### 9. No batching or templatizing without permission
Inherits the Global "No batching or templatizing" principle. Project-specific addition:
- This applies with particular force to repeated wireframe/screen generation — never generate multiple wireframes or screens as a batch without asking first, regardless of how repetitive or mechanical the pattern looks.

### 10. No deviation from the given task
- Execute the task as specified. Do not substitute your own approach, add unrequested scope, or reinterpret the goal.
- If you believe a deviation would produce a better outcome, propose it and get explicit approval before proceeding. Judgment is not authorization to act.

### 11. Binding documents — ADRs, specs, rule files
- ADRs, specification files, and rule files are binding constraints, not suggestions.
- Before starting a task, check for applicable ADRs/specs/rules and confirm compliance.
- If a request conflicts with an existing ADR or spec, flag the conflict to the user — do not silently override the document, and do not silently override the request.

### 12. Guardrails, gates, and zero gate-jumping
Inherits the Global "Zero gate-jumping / strict scope adherence" principle in full. Project-specific addition:
- Defined checkpoints in the GStack pipeline, wireframe/screen approval steps, and any other project workflow gate are hard stops.
- Do not skip, auto-approve, or work around a gate — including when you judge it unnecessary for a given task.

### 13. Checklist integrity — no rubber-stamping
Inherits the Global "Checklist integrity" principle in full. Project-specific addition:
- Applies specifically to per-wireframe and per-screen compliance checklists in `wireframes/` and `screens/`.
- When work repeats across multiple artifacts (e.g. 50 wireframes), apply the checklist fresh to each one — a pass on one wireframe is never evidence of a pass on another, and checklist results may not be copied, assumed, or bulk-applied across artifacts.
- Checklist verification happens within the owning skill's process — it is subject to Rule 8 (Skill Ownership) and must not be taken over by OpenCode.

### 14. Default-to-ask
Inherits the Global "Default-to-ask" principle in full — applies to skill selection, gate satisfaction, ADR/spec applicability, scope, and batching/templatizing decisions in this project.

---

## Workspace structure and file placement

| Folder | Purpose |
|---|---|
| `skill/` | PDLF skill definition files (SKILL.md and variants) |
| `docs/` | Durable PDLF methodology, system description, guides, and reference |
| `docs/adr/` | Architecture Decision Records for the PDLF framework |
| `docs/agents/` | Matt Pocock skills agent configuration |
| `docs/pre-pdlf/` | Pre-development research and reference materials |
| `wireframes/` | HTML wireframes, mockups, and visual layout references |
| `screens/` | Screen HTML mockups and their compliance checklists |
| `wip/` | Active work-in-progress: current plans, prompts, and files being prepared for delivery |
| `workspace/gstack/` | GStack working files and configuration |
| `workspace/gbrain/` | GBrain working files and configuration |
| `gstack-outputs/` | GStack-produced reports, reviews, task lists, and artifacts |
| `opencode-outputs/` | OpenCode-produced reports, reviews, and artifacts |
| `matt-skills-output/` | Matt Pocock skill outputs (copies — originals may also go to `%TEMP%`) |

---

## Artifact governance

**Principle:** Every session-produced artifact gets saved. The destination depends on who produced it and what kind of file it is. Never ask whether to save — always save (all such saves are Build actions; see Rule 5). Confirm the save location in the on-screen response.

- GStack often saves documents to a system-defined location. That behavior stays as-is, but a copy of every artifact or report GStack produces must **also** be saved to `gstack-outputs/`.
- Matt Pocock skills often save documents to a system-defined location. That behavior stays as-is, but a copy of every artifact or report a Matt Pocock skill produces must **also** be saved to `matt-skills-output/` (subject to the per-skill rules below).
- OpenCode-produced reports, reviews, and artifacts (i.e. produced directly, not via a skill) must be saved to `opencode-outputs/`.

### Routing by producer

| Producer | Destination |
|---|---|
| GStack (any gstack-* skill, including OpenClaw variants) | `gstack-outputs/` |
| OpenCode (direct, not via a skill) | `opencode-outputs/` |
| Matt Pocock skills | `matt-skills-output/` (see per-skill rules below) |

### Routing by file type (overrides producer routing)

These file types have fixed homes regardless of producer, and do not belong in a producer's output folder:

| File type | Destination |
|---|---|
| Durable documentation | `docs/` |
| Architecture Decision Records | `docs/adr/` |
| Wireframes and visual mockups | `wireframes/` |
| Screen HTML mockups | `screens/` |
| Wireframe/screen compliance reports | Same folder as the wireframe or screen it belongs to |
| DESIGN.md | `docs/` |

Every task, regardless of who performs it, must produce a task completion report stating what was done. The report is saved in the producer's output folder.

### Exclusions

This routing applies to reports, reviews, analyses, task lists, and similar session-produced artifacts. It does **not** apply to:
- Durable docs (`docs/`), ADRs (`docs/adr/`), wireframes (`wireframes/`), screens (`screens/`)
- Files in `wip/` (own lifecycle)
- GBrain working files (`workspace/gbrain/`)
- GStack working files (`workspace/gstack/`)

### Confirmation rule

The on-screen response must confirm the save location explicitly:
`"Saved to: gstack-outputs/<filename>."` or `"Saved to: opencode-outputs/<filename>."`
If the save fails, state why — do not silently report success.

### Promotion rules

- Long-lived specifications must not stay in `gstack-outputs/` — promote to `docs/`.
- Transient planning reports must not go in `docs/` unless explicitly promoted.
- ADRs must not go in `docs/` or `gstack-outputs/` — they belong in `docs/adr/`.

---

## GStack binary path override

When any GStack skill needs to call `gstack-slug`, `gstack-config`, or any other `gstack-*` binary, always use `/mnt/c/mybizz/gstack/bin/<binary-name>` directly. Do not attempt host-detection logic (`~/.codex/skills/gstack`, `.agents/skills/gstack`, or similar) to locate these binaries — that detection has been confirmed to fail in this environment. The literal path above is correct and verified.

---

## Matt Pocock skills artifact governance

> > **Verified 2026-07-06** — rules confirmed against actual SKILL.md files and project setup.

- `domain-modeling`, `grill-with-docs` → context output goes to `CONTEXT.md` at the project root. ADR output goes to `adr/` (overriding these skills' own documented default of `docs/adr/`).
- `to-prd`, `to-issues`, `triage` → leave at their existing default location, `.scratch/`. No redirect.
- `triage` rejection records → leave at their existing default, `.out-of-scope/`. No redirect.
- `setup-matt-pocock-skills` → leave its config output at its existing defaults, `docs/agents/` and `CLAUDE.md`. No redirect.
- `handoff`, `improve-codebase-architecture` → keep the default save to `%TEMP%` intact. ADDITIONALLY save a copy to `matt-skills-output/`. Never skip the `%TEMP%` save — only add to it.
- `teach` → redirect entirely to `C:\mybizz\matt-skills-teach` (outside this repo), not the skill's own default of the invoking directory.
- Any Matt Pocock skill not named above → no rule yet. Do not guess; ask the user before saving anywhere.

---

## Source of truth for this project

### Authoritative sources

- `skill/SKILL.md` — the canonical PDLF skill definition (and its active version as designated by the user)
- `docs/adr/` — architectural decisions for the PDLF framework
- `docs/` — PDLF methodology documentation and specifications
- `docs/pre-pdlf/` — pre-development research and reference materials
- The target project's own authoritative sources, as defined by that project's `AGENTS.md`, when performing work on or for a PDLF-managed project

### Fact rule

Do not use general training data or prior memory to assert project-specific facts. If a fact is not supported by the allowed files above, treat it as inferred or missing, not as known truth.

Research from GBrain must be validated before use — it cannot be depended on as a source of truth by itself.

### Exception handling

If the user explicitly instructs you to read a file outside the authoritative sources, you may use it only for that request and must state that its content is advisory unless the user explicitly promotes it.

### Mandatory reading

Any content given to you as added context with specified files must be treated as mandatory reading before proceeding.

---

## Certainty levels (must be explicit)

When assessing how well something is defined, every statement must fall into one of:

- `Clearly defined:` an explicit statement in an authoritative file directly answers the point.
- `Partially defined:` files give some constraints but important details are missing or implied.
- `Inferred only:` connecting dots not explicitly stated; depends on interpretation.
- `Missing:` the authoritative files do not provide enough information to answer reliably.

Do not upgrade inferred material to clearly defined.

---

## Language restrictions

### Schema and alignment claims
Do not claim the database schema "matches" or "fully implements" any document unless you were explicitly asked to compare them in this session and produced a concrete comparison output. Use cautious language: "broadly aligned with," "appears consistent with," "implements many of the patterns described in."

### Flow completeness claims
Do not use phrases like "fully specified" or "complete user flows" unless the files actually contain step-by-step or screen-by-screen flow descriptions.

---

## Access model

Governed by Global `AGENTS.md` at full strength — see Hard Rule 5 above. Summary as applied to this project:

### Plan agent
- May read the entire dev-pdlf repo and any PDLF-managed project repos for context.
- Does not write anywhere, in any repo, under any project-level setting.
- May propose plans, drafts, and content for Build to write, but does not write them itself.

### Build agent
- Is the only agent that writes in dev-pdlf, and only where the user has approved the specific change.
- Must not write in a PDLF-managed project's code repo except where Hard Rule 7 has been satisfied (coding phase active, and explicit case-by-case permission granted for that specific write).

---

## Safety rules, communication style

This project inherits the Global `AGENTS.md` Safety Rules and Communication Rules in full, with no local additions or exceptions. See Global `AGENTS.md` for the authoritative text — do not maintain a duplicate copy here.

---

## Prescribed standards

- Anvil.works working methods and standards are hard standards throughout. No deviation may ever be made without requesting authority and providing full justification for the deviation.
- Anvil.works uses the Material 3 theme, the M3 dependency, and the MUI overlay. These must be strictly adhered to at all times. No deviation is permitted.
- Color usage under this standard is governed by Hard Rule 2 (No hardcoded colors) — see `docs/DESIGN_palette_section.md` for token definitions.

---

## Agent skills

### Issue tracker
Issues are tracked in GitHub Issues on this repo. External PRs are not a triage surface. See `docs/agents/issue-tracker.md`.

### Triage labels
Uses the default five canonical labels: needs-triage, needs-info, ready-for-agent, ready-for-human, wontfix. See `docs/agents/triage-labels.md`.

### Domain docs
Single-context layout: one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

---

## Task completion reporting and audit trail

Inherits the Global "Task completion reporting," "Task completion prompts," and "Audit trail"
rules in full. Project-specific mechanics:

- Every task completion report is saved as its own `.md` file in the producer's output folder
  (`gstack-outputs/`, `opencode-outputs/`, or `matt-skills-output/` per Artifact Governance above).
- `devlog-index.md` (project root) references every task completion report by filename, in
  order, as the single index into the full history.
- `judgement-trail.md` (project root) collects, per task: issues flagged, developer decisions
  given, and any deviations approved. Extracted from each completion report at the time it is
  written — not reconstructed after the fact.

### Scaffold initialization

If `devlog-index.md` or `judgement-trail.md` does not exist when first needed, create it with
only the following header, then proceed:

`devlog-index.md`:
```
# Devlog Index
Reverse-chronological list of task completion reports.
```

`judgement-trail.md`:
```
# Judgement Trail
Flagged issues, developer decisions, and approved deviations, extracted per task.
```

This is a one-time structural creation, not a content decision — the agent does not invent
format beyond this header. Creating these two files is the one exception to the Plan/Build write
lock (Hard Rule 5) and the no-unrequested-action rules: this AGENTS.md rule is itself the standing
approval for that specific bootstrap action only. It does not authorize scaffolding anything else
unprompted.

---

## Change log

- Merged all content from `mb-3-cs/project-library/AGENTS.md` that is not strictly mb-3-cs-specific (the Mybizz Consulting Services project-purpose paragraph only — all other content is generalizable).
- Added Definitions section (OpenCode, Skill, Task, Gate, Artifact).
- Added Precedence order tier 6 (a skill's own default behavior) and conflict-trigger clause.
- Expanded Hard Rules from 5 to 14: added anvil.yaml protection, no hardcoded colors, no batching/templatizing, no deviation from given task, binding documents, guardrails/gates/zero gate-jumping, checklist integrity, and more detailed versions of Plan/Build writes, no-writing-to-managed-projects, default-to-ask. Merged mb-3-cs's no-time-references rule with dev-pdlf's existing documentation-standards rule.
- Separated no-time-references into its own rule (3) and documentation standards into its own rule (4) — both are individually gated and independently actionable.
- Updated Project purpose to explain `dev-` prefix and migration to `C:\pdlf\`, and added a sentence clarifying that all project-level references in this file are generic.
- Added Artifact Governance section: routing by producer, routing by file type, exclusions, confirmation rule, promotion rules.
- Added GStack binary path override.
- Added Matt Pocock skills artifact governance with per-skill routing.
- Added Source of truth, Certainty levels, and Language restrictions sections.
- Added Access model with Plan/Build boundaries.
- Added Prescribed standards section (Anvil.works, M3, MUI).
- Added Task completion reporting and audit trail with `devlog-index.md` and `judgement-trail.md` mechanics.
- Added Scaffold initialization clause for one-time auto-creation of `devlog-index.md` and `judgement-trail.md`.
- Trimmed Safety Rules and Communication Style to a pointer at the Global file, removing duplicated text.
- Workspace structure expanded with `wireframes/`, `screens/`, and `wip/`.
