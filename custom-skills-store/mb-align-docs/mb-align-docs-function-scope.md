---
document: mb-align-docs Function Scope
status: locked
version: 1
date: 2026-07-24
---

# mb-align-docs — Function Scope (Locked)

Companion skill to Docs Manager. Docs Manager handles folder/file structure.
mb-align-docs handles document *content* agreeing with itself — terminology,
requirements, cross-references, and document identity/lifecycle. The two
skills do not interoperate; they are separate entities with separate concerns.

This document locks down **function** — what the skill exists to do and the
rules governing that. **Framework** (safety nets, backup, resume/checkpoint,
approval mechanics) is a separate discussion, to follow.

---

## 1. Two distinct operations

mb-align-docs performs two operations that must never share a remediation
path:

### 1.1 Reference healing
A link points somewhere and the target moved, was renamed, or vanished.
- Local and frequent.
- Scoped to one target folder at a time — never work two target folders
  concurrently.
- Fix = repoint the reference to the correct current target.

### 1.2 Identifier change
The canonical name/identity of a *thing itself* is changing (not just a
broken pointer to it).
- Global in effect, rare.
- Cannot be handled as a local fix scoped to whatever folder happens to be
  under review — that produces silent, uncoordinated renames and
  re-invalidates references elsewhere (see: ADR numbering/prefix case).
- Any remediation that amounts to "this identifier is changing" — rather
  than "this pointer is wrong" — is flagged as an identifier-change event,
  not applied as a local fix.
- On approval, triggers an immediate, exhaustive, corpus-wide sweep for the
  old identifier in that session. Not queued, not deferred.
- This is the one explicit exception to the one-target-folder rule.

---

## 2. Directionality rules

- **Global → Global:** permitted (e.g. specifications-global may reference
  adr-global).
- **Global → Local:** never permitted. Global content must remain
  context-free and stable; it cannot depend on anything inside a specific
  project's lifecycle.
- **Local → Global:** permitted. Global is treated as stable and
  authoritative; local files reference it "blind."
- **Local → Local, same project:** permitted.
- **Local → Local, cross-project:** never live. See Section 3.

---

## 3. Cross-project references and the Quarantine mechanism

A live reference from one project into another project's local files is not
permitted. Rule, not a lean — uncommon occurrence, high potential to break a
project if left unmanaged.

Where content from another project (or a retired version of the same
project's own content) is genuinely wanted:

- It is **copied**, not linked, into a **Quarantine** location.
- The copy carries provenance in its front matter (source, date copied).
- Content in Quarantine may be viewed and cited in prose by a human, but
  **no trackable reference (front-matter reference, cross-doc link, or
  anything mb-align-docs parses and validates) may resolve to a target in
  Quarantine.** Same rule as for the Obsolete folder.
- Any trackable reference found pointing into Quarantine or Obsolete is a
  drift item: must be healed (repointed to the live successor) or removed.
  No third option.
- Purpose of Quarantine specifically: preserve content that would otherwise
  be lost when Obsolete is eventually purged. Quarantine and Obsolete are
  functionally identical to mb-align-docs (both terminal, both
  non-referenceable) — they differ only in survival, which is not this
  skill's concern.

---

## 4. Document identity: front matter

Front matter is **identity only**. It never carries references,
cross-references, or relationship data of any kind — that is determined
solely by what a document actually references in its body text. This is a
hard rule; it keeps reference-checking single-source-of-truth.

Front matter is set once, at document creation, and **never edited**. To
change what a document says or means is to create a new version — a new
document, with its own new, permanent front matter — not a mutation of the
old one.

Front matter includes a **state** field:
- `Live`
- `Retired`
- `Quarantine`
- (Obsolete is folder-based, outside front matter scope — tbc in framework)

A reference target's state field is authoritative for whether a reference to
it is valid. Any reference to a non-`Live` target is automatically a drift
item.

---

## 5. Versioning and retirement

- Only one version of a document is active (`Live`) at any time.
- Superseding a document means creating V2 (V3, etc.) as a new document with
  its own front matter. V1 does not get edited into V2.
- On supersession, the prior version's state flips to `Retired`.
- **No live reference to a retired document is permitted, ever.** There is
  no such thing as an "intentionally historical" reference in the live
  corpus. Either:
  - the reference is updated to point to the current live version, or
  - the retired document moves to Quarantine and the reference is removed
    (per Section 3).
- We never reference anything historical from within the live corpus.

---

## 6. Registers vs. index

Two different temporal orientations on similar-looking data — intentionally
kept separate:

- **Index** — what exists *right now*. Disposable, regenerable each run,
  current-state-only. This is Docs Manager's territory (e.g. `DOC-INDEX.md`
  in the prior skill iteration). mb-align-docs does not own this.
- **Register** — definitive record from creation to now. Append-only.
  Every entry is permanent; entries never disappear. A document's row
  carries an active/retired-style badge that updates, but the row itself is
  never removed.

### Register structure
- One register per project (`{slug}-project-library`).
- One register for `project-library-global`.
- Both include a **folder** column, since each of these corpora spans many
  folders.
- Registers persist between runs — never wiped/regenerated fresh, since
  there would be nothing left to validate against.

### Global register enforcement (hardcore, not advisory)
The `project-library-global` register is **enforced**, not merely
documentary:

- Every mb-align-docs run diffs the actual state of the global folders
  against the global register.
- The register always wins in any mismatch.
- A new global file not yet in the register gets added (append-only growth).
- A global file found renamed, or otherwise diverging from its registered
  identity, produces a mandatory developer decision: **rename back to match
  the register, or retire it.** No third option other than aborting the run
  entirely (which defeats the purpose and is not a real alternative).
- Once a global file is named, that name is permanent. To "rename" a global
  concept, a new document is created under a new name and separately
  registered (consistent with Section 5 versioning rules) — the old one
  retires, it is not overwritten.

---

## 7. Explicit separation from Docs Manager

Docs Manager and mb-align-docs serve different purposes and do not
interrelate or depend on one another:
- Docs Manager: folder/file structure, current-state index.
- mb-align-docs: document content agreement, cross-references, document
  identity/lifecycle, register.

Neither leans on the other's outputs as an input. Each stands alone.

## 8. UI documentation model: wireframe-definition / ui-scaffold

Per form-folder under a project's `ui\` folder
(`ui\{package-name}\{form-name}\`), exactly six files exist, with a
three-file/three-file front-matter split:

**Documents with front matter (authored, in scope):**
- `{form-name}-wireframe-definition.md` — derives meaningfully from
  architecture; has its own identity and lifecycle.
- `{form-name}-wireframe-chklist.md` — filled in/marked per form during
  build, making it an applied record, not a static template.
- `{form-name}-screen-chklist.md` — same reasoning as
  wireframe-chklist above.

**Build artefacts without front matter (permanently out of scope):**
- `{form-name}-wireframe.html` — HTML output, not an authored document.
- `{form-name}-screen.html` — HTML output, not an authored document.
- `{form-name}-screen.png` — image output, not a document at all.

The three HTML/PNG files are permanently out of front-matter scope by
nature — "no front matter" on these is correct and expected, never a gap
to flag on future Detect passes. The three markdown files do get front
matter and register entries like any other in-scope document.

---

## Status
Function scope agreed and locked as of 2026-07-24. Framework (safety nets,
backup-before-write, resume/checkpoint, approval mechanics, naming) is the
next discussion, building on the same disciplines already established in
Docs Manager and V1 of this skill.
