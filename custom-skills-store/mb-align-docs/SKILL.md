---
document: mb-align-docs SKILL
version: V1
status: active
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

Manual only. `/mb-align-docs` — developer-triggered, no autonomous
scheduling. Run against one project at a time.

## Scope rule

Every run targets exactly one project's `{slug}-project-library`, **plus**
`project-library-global` unconditionally — global is always in scope
alongside whichever project was named, never run on its own and never
excluded. This is because local↔global directionality checks (function
scope Section 2) require both sides present in the same run to validate.

## Subagents

- **detector** — read-only. Performs Detect. Cannot write.
- **writer-text** — write permission for text-level corrections only:
  reference repoints (healing fixes). Cannot rename, move, or retire a
  file.
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
| `...\learnings\` | Judgment calls from past runs |
| `...\in-progress\` | Current run only — resume source if interrupted |
| `...\last-completed-run\` | Full detail of most recent finished run |
| `...\abandoned-runs\` | Declined-resume runs, kept, never deleted |
| `C:\backups-general\backup-mb-align-docs_<timestamp>\` | Verified backups, one snapshot per run |
| `{slug}-project-library\register-local\` | Project's populated register |
| `project-library-global\register-global\` | Global populated register |
| `{project}\Quarantine\`, global `Quarantine\` | Quarantined copies |
