---
document: Register Template
version: V1
status: template
---

# Register — {PROJECT-NAME or project-library-global}

Append-only. Rows are never deleted. Only `state`, `date-state-changed`,
`superseded-by`, and `notes` may be updated in place after a row is created.
All other fields are permanent from first registration.

| doc-id | filename | folder | type | state | date-registered | date-state-changed | superseded-by | notes |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

## Field definitions

- **doc-id** — permanent identifier, assigned at first registration, never reused even if the row's state becomes Retired.
- **filename** — current registered name of the document.
- **folder** — folder within the corpus this document lives in.
- **type** — document type (adr / spec / policy / guide / checklist / sop / template / other).
- **state** — one of: `Live`, `Retired`, `Quarantine`. Mirrors the document's own front-matter `state` field.
- **date-registered** — date this row was first appended. Never changes. Format:
  `YYYY-MM-DDTHHMMSS+0200` (Filename-Safe ISO 8601, UTC+2), per `global-0037` (C:\dev\project-library-global\adr-global\timezone-utc-storage-display-conversion.md).
- **date-state-changed** — date `state` last changed. Updated whenever state flips.
  Same format as date-registered.
- **superseded-by** — doc-id of the version that replaced this one, if state is Retired due to versioning. Blank otherwise.
- **notes** — free text, optional.
