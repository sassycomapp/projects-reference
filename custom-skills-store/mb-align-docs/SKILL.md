---
document: mb-align-docs SKILL
version: V1
status: active
triggers:
  - /mb-align-docs
  - /mb-align-docs <slug>
---

# mb-align-docs — SKILL

Document-consistency checker. Cross-references every document in scope
against every other, finds terminology mismatches, contradicting
requirements, stale references, orphaned cross-references, and identifier
drift, then walks the developer through findings one at a time with an
opinionated recommendation, fixing only what is approved.

Companion to Docs Manager. Does not interoperate with Docs Manager — see
`mb-align-docs-function-scope.md` Section 7. Docs Manager owns folder/file
structure and current-state indexing. This skill owns document *content*
agreement, cross-references, and document identity/lifecycle. Framework
below deliberately mirrors Docs Manager's proven structure.

Full function rules: `mb-align-docs-function-scope.md`.
Finding structure: `finding-report-format.md`.
Log structure: `alignment-log-template.md`.
Front matter rules: `front-matter-schema.md`.
Register structure: `register-template.md`.
Quarantine provenance: `quarantine-provenance-template.md`.
Short-form summary: `README.md`.

## Invocation

Manual only. `/mb-align-docs <slug>` — developer-triggered, no autonomous
scheduling. `<slug>` names the project (matching
`{slug}-project-library`). Run against one project at a time.

## Scope rule

Every run targets exactly one project's `{slug}-project-library`, **plus**
`project-library-global` unconditionally — global is always in scope
alongside whichever project was named, never run on its own and never
excluded. This is because local↔global directionality checks (function
scope Section 2) require both sides present in the same run to validate.

## Subagents

- **detector** — read-only. Performs Detect. Cannot write.
- **writer-text** — write permission for text-level corrections only:
  reference repoints (healing fixes) and front-matter insertion for
  missing-front-matter and incomplete-front-matter findings. Cannot
  rename, move, or retire a file.
- **writer-structural** — write permission for structural actions only:
  identifier-change renames plus their triggered sweep, register
  rename/retire actions, Quarantine moves. Requires the second
  confirmation gate (below) before executing, in addition to standard
  approval.
- **logger** — append-only. Performs Commit/Log and Learnings write-back.

Split mirrors Docs Manager's `apply` (text) vs `filesystem` (renames/
moves) separation — different risk categories get different write
permissions, never a single subagent authorized for both.

## Phase sequence

−2. **Resume check** — on invocation, check
    `C:\mybizz\logs\mb-align-docs\in-progress\` for an unfinished run
    against the target project. If found, offer to resume from exactly
    where it left off, including decisions already made, instead of
    starting over. If declined, move that run to
    `C:\mybizz\logs\mb-align-docs\abandoned-runs\` (never deleted) and
    start fresh.
−1. **Learnings (open)** — load
    `C:\mybizz\logs\mb-align-docs\learnings\` for both the target project
    and `project-library-global`. Prior corrections steer this run.
0. **Precondition + Backup** — git working tree must be clean for the
   target project and for `project-library-global`; stop immediately and
   report if either is dirty. Then take a verified backup to
   `C:\backups-general\backup-mb-align-docs_<timestamp>\`, where
   `<timestamp>` is `YYYY-MM-DDTHHMMSS+0200` (Filename-Safe ISO 8601,
   UTC+2, per `global-0037`, C:\dev\project-library-global\adr-global\timezone-utc-storage-display-conversion.md). Run stops cold if the backup cannot be
   verified — no exceptions, mirrors Docs Manager.
1. **Detect** (`detector`) — scan the declared target folder within the
   named project, or, for an identifier-change sweep, the full corpus
   (project + global together). Read-only. Also runs the register
   enforcement diff (see below). Produces findings per
   `finding-report-format.md`. Progress written to
   `in-progress\` as it goes, so a session dying here can be resumed.
2. **Report** (`detector`) — presents every finding with an opinionated
   recommendation. Never a bare list, never unrecommended.
3. **Approve** — developer approves per finding, individually. No batch
   approval across unrelated findings.
4a. **Apply — text** (`writer-text`) — writes approved reference-healing
    repoints only.
4b. **Apply — structural** (`writer-structural`) — a second explicit
    "about to do this, proceed?" confirmation, on top of the Phase 3
    approval, before executing any identifier-change rename/sweep,
    register rename/retire, or Quarantine move.
5. **Verify** (`writer-text` / `writer-structural`) — re-read what was
   just written; confirm it resolved the approved finding before moving
   to the next.
6. **Commit / Log** (`logger`) — append-only alignment-log entry per
   change; commit to git for both the project and global repos as
   applicable.
7. **Learnings (close)** (`logger`) — write back anything notable for
   next run's Phase −1. Move this run's record from `in-progress\` to
   `last-completed-run\`.

## Hard constraints

- No writes without approval — Detect and Report never touch a file.
- Structural writes require approval *plus* the Phase 4b confirmation.
- Mandatory verified backup before any write phase — run stops cold if
  backup cannot be verified.
- Git-clean precondition on both the target project and global — refuse
  to run Detect if either is dirty.
- No deletion of content — heal, retire, or quarantine only. Emptying
  Quarantine or Obsolete is out of scope for this skill.
- No autonomous state-changing commands — `detector` cannot invoke either
  writer; a write only occurs following explicit developer approval (and,
  for structural actions, the second confirmation) of a specific finding.
- One target folder at a time within a run, except an approved
  identifier-change finding, which triggers an immediate corpus-wide
  sweep (project + global) within the same session, visibly, before
  Apply proceeds.
- A lost session never loses work — resume picks up exactly where it left
  off, including prior decisions in that run.

## Two operations — never share a remediation path

- **Reference healing** — local, frequent, one folder at a time. Routes to
  `writer-text`.
- **Identifier change** — global in effect, rare. Routes to
  `writer-structural`. Triggers corpus-wide sweep on approval.

See `mb-align-docs-function-scope.md` Section 1.

## Register enforcement gate

Runs as part of Detect, after Backup/precondition:

- Diff live `project-library-global` folders against the populated global
  register at `project-library-global\register-global\`.
- Register always wins on mismatch.
- New unregistered global file → appended to register on approval.
- Renamed/diverged global file → exactly two options: rename-to-match-
  register, or retire. Routes to `writer-structural`, requires Phase 4b
  confirmation. No bypass except aborting the entire run.

See `mb-align-docs-function-scope.md` Section 6.

## Missing-front-matter check

Runs as part of Detect, after Backup/precondition:

- Re-read every in-scope document (per the skill's existing
  scope/exclusion rules) to check for the presence of a YAML
  front-matter block (--- delimited).
- A document found without front matter is a distinct finding type:
  "missing front matter."
- For each bare document, the detector proposes an identity:
  document (file path), doc-id (determined from path per
  project-inventory), state (proposed as Live), date-created (file
  creation date or first git commit for the file).
- Presented for developer approval during Phase 3 (Approve), same as
  a register-mismatch finding. Missing-front-matter findings across
  the run may be batch-approved as a single group — they share the
  same finding type and remediation path, so they are not "unrelated
  findings" under the Phase 3 rule.
- On approval, Apply writes the front-matter block using the approved
  identity. Routes to writer-text for the write action — this is a
  content addition, not a structural rename, move, or retire.
- After writing, Verify re-reads the document to confirm the block
  is present and matches the approved identity before moving to the
  next.
- This check does not alter what counts as in-scope or out-of-scope;
  it only detects documents already in scope that lack front matter.

- A document that has a front-matter block but is missing one or more of
  the four required fields (document, doc-id, state, date-created, per
  front-matter-schema.md) is a distinct finding type: "incomplete front
  matter." This is separate from "missing front matter" (no block at all).

- For each incomplete front-matter document, the detector proposes only
  the missing field(s) — fields already present and correct are left
  untouched.

- Presented for developer approval during Phase 3 (Approve). Incomplete-
  front-matter findings across the run may be batch-approved as a single
  group — they share the same finding type and remediation path, so they
  are not "unrelated findings" under the Phase 3 rule.

- On approval, Apply writes only the missing field(s) into the existing
  block, preserving all pre-existing fields as-is. Routes to writer-text
  for the write action — this is a content addition to existing front
  matter, not a structural rename, move, or retire.

- After writing, Verify re-reads the document to confirm the block now
  contains all four required fields, and that no pre-existing field was
  altered.
- Content under `templates-global` and `templates-local` folders is
  excluded from this check entirely — templates are reusable stencils,
  not documents with identity or lifecycle, and are never expected to
  carry front matter. This is a categorical exclusion, not a recurring
  gap to flag.
- Files named `devlog-index.md` or `judgement-trail.md`, anywhere within
  a project-library, are historical logs and excluded from front-matter
  scope — "no front matter" is correct and expected for these files, by
  design, regardless of which folder they sit in.

## Directionality rules (enforced during Detect)

- Global → Global: permitted.
- Global → Local: never permitted.
- Local → Global: permitted.
- Local → Local, same project: permitted.
- Local → Local, cross-project: never live — must be a Quarantine copy
  with provenance. Routes to `writer-structural`.

## Versioning rule (enforced during Detect)

No live reference may resolve to a document whose front-matter `state` is
not `Live`. A reference to a `Retired` or `Quarantine` document is always
a drift item — heal to the current live version, or remove.

## Folders

| Folder | Holds |
|---|---|
| `C:\mybizz\logs\mb-align-docs\` | Permanent history, one record per run |
| `C:\mybizz\logs\mb-align-docs\learnings\` | Judgment calls from past runs |
| `C:\mybizz\logs\mb-align-docs\in-progress\` | Current run only — resume source if interrupted |
| `C:\mybizz\logs\mb-align-docs\last-completed-run\` | Full detail of most recent finished run |
| `C:\mybizz\logs\mb-align-docs\abandoned-runs\` | Declined-resume runs, kept, never deleted |
| `C:\backups-general\backup-mb-align-docs_<timestamp>\` | Verified backups, one snapshot per run |
| `{slug}-project-library\register-local\` | Project's populated register |
| `project-library-global\register-global\` | Global populated register |
| `{project}\Quarantine\`, global `Quarantine\` | Quarantined copies |
| `{slug}-project-library\ui\{package}\{form}\` | Per-form UI documentation: wireframe-definition, checklists (authored, with front matter), plus wireframe/screen HTML and PNG artefacts (build output, permanently out of front-matter scope) |
