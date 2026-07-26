---
document: Front Matter Schema
version: V1
status: reference
---

# Front Matter Schema — V1

Front matter is identity only. It never carries references, cross-references,
or relationship data — those are determined solely by what a document
actually references in its body text.

Front matter is set once, at document creation, and never edited. To change
what a document says or means, create a new version (new document, new
front matter). See mb-align-docs Function Scope, Section 5.

## Required fields

- **document** — the document's title/name.
- **doc-id** — matches the doc-id assigned in the relevant register on
  first registration.
- **state** — one of: `Live`, `Retired`, `Quarantine`. Authoritative for
  whether references to this document are valid. Any reference to a
  non-Live target is a drift item.
- **date-created** — date this document (this specific version) was created. Format:
  `YYYY-MM-DDTHHMMSS+0200` (Filename-Safe ISO 8601, UTC+2), per the timestamp standard
  in `global-0037` (C:\dev\project-library-global\adr-global\timezone-utc-storage-display-conversion.md; Client Timezone / Build Artefact Timestamp Format Standard). A
  date-only value (`YYYY-MM-DD`) is acceptable where an internal document date is being
  carried over and only a date, not a time, was originally stated.

## Conditional fields

- **superseded-by** — doc-id of the version that replaced this one. Required
  if state is Retired due to versioning; omitted otherwise.
- **source** — required only for documents in Quarantine that originated
  from another project or as a copy of a retired version. See
  quarantine-provenance-template-v1.md.

## Not permitted in front matter

- Reference lists, "see also" fields, cross-links of any kind.
- Any field describing what this document points to or is pointed to by.
  Reference-checking is done from body text only, never from front matter.
