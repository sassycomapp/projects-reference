---
name: align-docs
description: Sanity-check a whole library of planning and specification documents against each other and repair where they've drifted apart — terminology mismatches, contradicting requirements, stale references to concepts that changed elsewhere, orphaned cross-references. Use when the user asks to "align the docs", "realign the planning documents", "make sure everything ties back to everything else", "sanity check the specs", or has made significant edits to one document and is worried it broke consistency with the rest of the library. This is a many-documents-against-each-other sweep, not a single-document review — do not use for auditing one plan in isolation (that's plan-eng-review or plan-design-review) and not for syncing docs against shipped code (that's document-release).
---

# Align Docs

## Why this exists

Specs drift. You fix terminology in one document, add a constraint in another, and three weeks later nothing tells you the wireframe spec still calls something by its old name, or that two documents now disagree about how a feature should behave. Nobody reads the whole library end to end to catch this — it's tedious and error-prone by hand. This skill does that sweep, surfaces every place where documents disagree, and fixes them with your sign-off rather than silently.

## The drift map

The output of the cross-reference step (Step 3) is called the **drift map** — the complete set of contradictions, terminology mismatches, and stale references found across the document set. Every later step in this skill operates on the drift map. Refer to it by name once it exists, rather than re-describing "the set of inconsistencies" each time.

## Steps

### Step 1: Ingest

Identify the document set in scope. If the user names a folder or project (e.g., "Project Library"), read every planning/specification document in it. If scope is ambiguous, ask which documents or directories are in play before continuing — don't guess at scope for a sweep this size.

**Completion criterion:** every document in scope has been read in full.

### Step 2: Index

For each document, extract its claims, defined terms, and decisions into `DOC-INDEX.md` (template in `references/templates/doc-index-template.md`). A claim is anything another document could agree or disagree with: a named component, a stated requirement, a data shape, a decision about behavior. Don't summarize the document's prose — index its assertions.

If a `GLOSSARY.md` already exists (shared with the `domain-modeling` skill), check new terms against it rather than building a second glossary from scratch.

**Completion criterion:** every document has a corresponding entry in `DOC-INDEX.md`, and every defined term in it has been checked against `GLOSSARY.md` if one exists.

### Step 3: Cross-reference

Compare every document's index entry against every *other* document's index entry — not just neighboring or obviously-related pairs. Drift hides in documents that don't look related until you check. For each pair, look for:

- **Terminology mismatch** — the same concept named differently in two places, or two concepts sharing one name.
- **Contradicting requirement** — one document states a rule the other document's spec violates.
- **Stale reference** — a document points at a component, field, or decision that another document has since renamed, removed, or redefined.
- **Orphaned cross-reference** — a document references another document, section, or component that no longer exists in that form.

Log every hit into the drift map, not just the ones that look serious — a small terminology drift now is the contradiction that costs an afternoon later.

**Completion criterion:** every document pair has been compared, and the drift map lists every hit found, however minor.

### Step 4: Surface

Walk the user through the drift map one entry at a time. For each one, state plainly what conflicts with what, and give an opinionated recommendation for which side should win (same pattern as an architecture review: don't just list the problem, say what you'd do about it and why). Let the user override.

**Completion criterion:** every entry in the drift map has been shown to the user — none resolved silently in the background, none dropped because it seemed minor.

### Step 5: Resolve

For each entry the user wants fixed now, interview briefly to confirm the resolution — which document's version is correct, what the corrected text should say — then write the fix back into the source document(s). Never bulk-edit without this confirmation step; a wrong silent fix is worse than a flagged inconsistency, since it looks resolved when it isn't.

Entries the user defers stay logged as open in the drift map rather than disappearing.

**Completion criterion:** every drift map entry is marked either resolved (with the fix written back) or explicitly deferred — none left in an ambiguous state.

### Step 6: Report

Write an entry to `ALIGNMENT-LOG.md` (template in `references/templates/alignment-log-template.md`) recording what changed this run: which documents were touched, what was fixed, what was deferred and why. This is the realignment equivalent of an ADR — it lets a future run (or a future person) see what's already been settled instead of re-litigating it.

**Completion criterion:** a new entry exists in `ALIGNMENT-LOG.md` and accounts for every fix made and every deferral from this run.

## What counts as a conflict

Use this as the working definition throughout Steps 3–5:

| Type | What it looks like |
|---|---|
| Terminology mismatch | Two documents use different names for the same thing, or the same name for two different things |
| Contradicting requirement | One document's stated behavior or constraint is incompatible with another's |
| Stale reference | A document points at something (component, field, decision) that has since changed elsewhere |
| Orphaned cross-reference | A document references another section/document/component that no longer exists in that form |

## Don't duplicate, reuse

- For glossary/terminology discipline, defer to the `domain-modeling` skill's `GLOSSARY.md` rather than building a parallel one.
- For the resolve-by-interview mechanic in Step 5, use the same lightweight interview discipline as `grill-with-docs` rather than reinventing a question loop — a few sharp questions to confirm the fix, not a full grilling session.
