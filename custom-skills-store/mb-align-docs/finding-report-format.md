---
document: Finding Report Format
version: V1
status: reference
---

# Finding Report Format — V1

One consistent shape for every finding. Every finding must carry an
opinionated recommendation — never a bare problem statement, never a
neutral list of options for the developer to sort out unaided.

## Common fields — every finding

- **finding-id** — sequential within the run, for log cross-reference.
- **type** — one of: `reference-healing` | `identifier-change` |
  `register-mismatch` | `missing-front-matter` |
  `incomplete-front-matter`.
- **target** — the document(s) involved.
- **evidence** — what was actually found (the broken link, the diverging
  name, the register mismatch).
- **recommendation** — the opinionated fix, stated as a decision, not a
  menu of possibilities.
- **options** — the fixed, limited response set for this finding-type.
  Never open-ended.

## Type-specific behaviour

### reference-healing
Recommendation names the specific correct target.
**Options: Approve / Reject / Skip.**

### identifier-change
Recommendation states the new canonical identifier and explicitly notes
that approval triggers a corpus-wide sweep within this same session.
**Options: Approve (triggers sweep) / Reject.**

### register-mismatch
Per function scope Section 6 — exactly two live options, always.
**Options: Rename-to-match-register / Retire.**
No Skip, no Reject. Only abort-the-entire-run sits outside this, and that
is a run-level action, not a response to an individual finding.

### missing-front-matter
Recommendation proposes the full front-matter block (document, doc-id,
state, date-created), determined per SKILL.md's missing-front-matter
check.
**Options: Approve / Reject / Skip.**
Batch-approvable as a group per SKILL.md Phase 3 — findings share the
same type and remediation path.

### incomplete-front-matter
Recommendation proposes only the missing field(s) — fields already
present and correct are left untouched.
**Options: Approve / Reject / Skip.**
Batch-approvable as a group — same reasoning as missing-front-matter
(findings share the same type and remediation path).

## Rule

No finding is ever presented without a recommendation already attached.
Detection and Report are read-only — this format is produced by `detector`
and consumed by the developer at Approve; only `writer` acts on the
outcome.
