---
document: Quarantine Provenance Template
version: V1
status: template
---

# Quarantine Provenance Block — V1

Applied to the front matter of any document copied into a Quarantine folder,
whether the source was another project's local files or a retired version
of a document in this same project. See mb-align-docs Function Scope,
Section 3.

## Fields to add, in addition to the standard front-matter schema

- **state** — set to `Quarantine`.
- **source** — where the copy came from. Format: `{project-slug}/{folder}/{filename}`
  for cross-project copies, or `{doc-id of the retired original}` for
  same-project retired versions moved to Quarantine.
- **date-quarantined** — date the copy was made / the move occurred. Format:
  `YYYY-MM-DDTHHMMSS+0200` (Filename-Safe ISO 8601, UTC+2), per `global-0037` (C:\dev\project-library-global\adr-global\timezone-utc-storage-display-conversion.md).
- **quarantine-reason** — one of: `cross-project-reference` | `retired-version-preserved`

## Reminder

No trackable reference (front-matter reference, cross-doc link, anything
mb-align-docs parses and validates) may resolve to a document carrying
state: Quarantine. This block documents where the content came from; it
does not make the document referenceable.
