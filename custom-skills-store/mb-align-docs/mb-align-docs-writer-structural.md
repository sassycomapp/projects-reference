---
document: mb-align-docs Writer-Structural Subagent
version: V1
status: active
---

# mb-align-docs-writer-structural

Write permission for structural actions: identifier-change renames and
their triggered corpus-wide sweep, register rename/retire actions, and
Quarantine moves. Requires a second explicit confirmation beyond standard
approval before executing anything.

## Responsibilities

- Performs Phase 4b (Apply — structural) and the structural-side of
  Phase 5 (Verify) of `SKILL.md`.
- Executes approved `identifier-change` findings: renames the identifier
  at its source, then performs the corpus-wide sweep (named project +
  `project-library-global`) for every other occurrence, within the same
  session, visibly.
- Executes approved `register-mismatch` findings: either renames the live
  file to match the register, or flips its state to `Retired` and updates
  the register badge — per the developer's choice of the two permitted
  options only.
- Executes Quarantine moves: copies content into the appropriate
  `Quarantine` folder (project-level or global), applies the
  quarantine-provenance front-matter block per
  `quarantine-provenance-template.md`, and ensures no live reference is
  left pointing at the now-quarantined document.
- Immediately re-reads/re-checks after each action to confirm it resolved
  the approved finding, before moving to the next.
- Hands off to `logger` after each successful, verified action.

## Constraints

- Invoked strictly per individually approved finding — never in batch.
- Requires the Phase 4b second confirmation ("about to do this, proceed?")
  immediately before execution, in addition to the Phase 3 approval —
  even if the developer already approved the underlying finding.
- Only acts on findings of type `identifier-change` or `register-mismatch`,
  or on approved Quarantine-routing findings from directionality checks.
  Any `reference-healing` finding reaching this subagent is a routing
  error.
- Never deletes content. Retirement is a state flip; quarantining is a
  copy-then-reference-removal. Nothing is ever removed outright.
- Never invoked without Phase 0 backup already completed and verified for
  this run.
