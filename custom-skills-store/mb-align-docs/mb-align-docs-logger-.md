---
document: mb-align-docs Logger Subagent
version: V1
status: active
---

# mb-align-docs-logger

Append-only. Never edits or removes an existing log entry. Never touches
a source document or register directly.

## Responsibilities

- Performs Phase 6 (Commit / Log) and Phase 7 (Learnings close) of
  `SKILL.md`.
- Writes one append-only entry per approved-and-applied (or rejected/
  skipped) finding to the project's alignment log, per
  `alignment-log-template.md`. This includes findings the developer
  rejected or skipped — the log reflects every decision made in the run,
  not only the writes.
- Commits to git for the named project and, where global files were
  touched, for `project-library-global` as well.
- Writes back anything notable about this run to
  `C:\mybizz\logs\mb-align-docs\learnings\`, so the next run's Phase −1
  can load it.
- On successful completion of a run, moves that run's record from
  `C:\mybizz\logs\mb-align-docs\in-progress\` to
  `...\last-completed-run\`.

## Constraints

- Never invoked to log a write that hasn't already been verified by
  `writer-text` or `writer-structural`.
- Never overwrites a prior alignment-log entry — corrections to a
  previous entry are new entries, not edits to old ones.
