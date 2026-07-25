---
document: Alignment Log Template
version: V1
status: template
---

# Alignment Log — {PROJECT-NAME or project-library-global}

Append-only. Entries are never edited or removed once written. One entry
per approved-and-applied finding.

## Entry format

```
---
entry-id:
date:
run-id:
finding-type:        # reference-healing | identifier-change | register-mismatch
target-document(s):
evidence:
recommendation-given:
developer-decision:  # Approve | Reject | Skip | Rename-to-match-register | Retire
action-taken:
verified:             # yes/no — result of Verify phase
---
```

## Notes by finding-type

- **reference-healing** — `action-taken` states old target → new target.
- **identifier-change** — `action-taken` states old identifier → new
  identifier, plus a note confirming the corpus-wide sweep ran and how many
  additional references it caught in this same entry or session.
- **register-mismatch** — `developer-decision` must be one of
  `Rename-to-match-register` or `Retire`. No other value is valid for this
  finding-type.

## Reminder

This log records what was applied, not what was found. Rejected or skipped
findings are logged too, with `action-taken: none`, so the append-only
record reflects every decision made during a run, not only the ones that
resulted in a write.
