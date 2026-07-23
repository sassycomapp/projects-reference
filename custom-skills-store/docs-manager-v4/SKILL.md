---
name: docs-manager
description: |
  System document inventory curator. Keeps division maps, project inventories, INDEX files,
  READMEs, and AGENTS.md files synchronized with the actual filesystem. Runs the full
  learnings → backup → detect → report → approve → apply → verify → commit/push → log →
  learnings cycle. Scans all in-scope documents for stale internal cross-references and
  directory drift.

  Scope is exclusion-based: everything under C:\ is in scope except a defined blocklist
  (system folders, code repos, backup/archive/obsolete folders, tooling folders). New
  documents and new code repos are picked up automatically as the filesystem grows — no
  manual list maintenance required.

  Invoked when structural changes are made (folders created, renamed, moved, deleted)
  to ensure the navigational documents agents depend on stay current.
triggers:
  - /docs-manager
  - /docs-manager <scope>
---

# Skill: docs-manager

## Quick Reference

| Aspect | Value |
|---|---|
| **Command** | `/docs-manager`, `/docs-manager <scope>` |
| **Role** | System document inventory curator |
| **Trigger** | Manual only — developer invokes when structural changes have been made |
| **Scope model** | Exclusion-based (blocklist), not a curated allowlist |
| **Backup** | Mandatory, verified, automatic — precedes every scan, no approval needed |
| **Learnings** | Loaded before every run, updated after every run — overrules SKILL.md except Hard Constraints |
| **Approval** | Change report presented → developer approves → changes applied |
| **Commit** | Approved changes are committed and pushed to GitHub after verification |
| **Log** | `C:\mybizz\logs\docs-manager\<timestamp>.md` |
| **Orchestrator** | OpenCode, always |

---

## HARD CONSTRAINTS

These three rules cannot be overridden by anything — not a SKILL.md edit made casually, not a
Learnings entry, not a scope parameter. They are the floor the entire system stands on.

1. **Nothing is written, modified, or deleted anywhere, at any phase, except in Phase 4 — and
   Phase 4 only ever applies changes the developer explicitly approved in Phase 3.** Three
   phases are exempt from this because they only ever create brand-new files and never touch,
   modify, or delete an original file: **Phase 0 (Pre-flight Check & Backup)**, which checks git
   status and creates copies; **Phase 7 (Log)**, which writes a new timestamped log entry; and
   **Phase 8 (Learnings)**, which writes new learning files. No other phase may write anything
   under any circumstance.
2. **Phase 0 must run in full, in order, before Phase 1 (Scan) begins — every single
   invocation, no exceptions.** Phase 0 has two mandatory steps, both blocking: (a) `git status`
   on every in-scope Active Repository — if any is dirty, abort the entire run immediately with
   the `REPOS_DIRTY` list, before backup or scan; (b) backup the in-scope file set and verify
   the copy — if verification fails, abort with `BACKUP_FAILED`. Neither step may be skipped,
   reordered, or asked about — invocation of `/docs-manager` is what triggers both, immediately
   (Section 5).
3. **Docs Manager never runs `gbrain sync` or any GBrain command that changes state.** Only
   read-only GBrain inspection (e.g. `gbrain sources list`) is permitted, and only where Section
   3 explicitly calls for it.

---

## 0. Purpose

Keeps the system's navigational documents synchronized with reality. When folders are created,
renamed, moved, or deleted; when scripts are added; when the division hierarchy shifts — the
maps, indexes, and inventories that agents depend on must stay current. This skill runs the
full learnings → backup → detect → report → approve → apply → verify → commit/push → log →
learnings cycle.

Internal cross-references between documents are also verified — a document referencing a path
that no longer exists or has moved is flagged as an anomaly.

Scope is defined by exclusion, not inclusion: everything under `C:\` is considered in scope
unless it matches an exclusion rule (Section 1.2). This means new documents, new projects, and
new code repos enter scope automatically as they're created — the developer maintains the
exclusion rules, not a growing list of individual files.

The system also accumulates institutional memory over time (Section 4, Learnings) — judgment
calls the developer makes once don't need to be re-made from scratch on every future run.

---

## 1. Scope Definition

### 1.1 Walk Root

`C:\` (entire drive), minus everything in Section 1.2.

### 1.2 Exclusions

#### 1.2.1 Absolute Path Exclusions

Exact top-level matches — never entered, never walked:

```
C:\Windows
C:\appverifUI.dll
C:\vfcompat.dll
C:\$SysReset
C:\$WinREAgent
C:\.pnpm-store
C:\_ BIG BACKUP
C:\_BIG BACKUP 2
C:\_Data-not mybizz
C:\eSupport
C:\Intel
C:\MSOCache
C:\OneDriveTemp
C:\pdlf
C:\PerfLogs
C:\Program Files
C:\Program Files (x86)
C:\ProgramData
C:\python
C:\SSH
C:\temp
C:\Users
C:\mybizz\skills
C:\backup-docs-manager
```

`C:\mybizz\skills\` was added here after the `project-inventory.md`/`git-repo-inventory.md`
merge surfaced that it had no exclusion anywhere, unlike `gbrain`/`gstack` which happen to match
the name-pattern rule below. It's a third-party, not-user-managed tool repo (Section 3.1,
Installed Tools) — same category as `gbrain`/`gstack`, just with a name that doesn't match the
pattern, so it needed an explicit absolute-path entry instead.

#### 1.2.2 Config Directory Exclusions (recursive, by name, anywhere in scope)

Any directory with these names is excluded, wherever it appears inside an otherwise
in-scope folder:

```
.git
.opencode
```

#### 1.2.3 Code Repo Exclusion (structural rule)

A folder is a **code repo** — and its *contents are excluded from scanning*, including its
README — when **both** of the following are true:

1. **Naming convention**: it is an immediate child of `C:\dev\dev-<slug>\`, and its own folder
   name equals `<slug>` (the parent folder name with the `dev-` prefix stripped).
2. **Marker file**: it contains `anvil.yaml` at its root.

This is a structural rule, not a maintained list — new repos are detected automatically as
they're created, following the existing convention:

```
C:\dev\dev-mb-3-cs\mb-3-cs\        ← code repo (matches slug, has anvil.yaml)
C:\dev\dev-mb-3-cs\project-library\ ← NOT a code repo — stays in scope
```

**"Excluded" means content-excluded, not reference-invisible** (see Section 2.5) — a docmap
that lists a code repo folder is still checked to confirm that folder exists on disk. Docs
Manager just never opens, globs inside, or reads anything belonging to the repo itself.

**Mismatch handling (never guess):**

| Condition | Classification | Action |
|---|---|---|
| Name matches convention + `anvil.yaml` present | Confirmed code repo | Content-excluded silently |
| Name matches convention, `anvil.yaml` **missing** | `CODE_REPO_AMBIGUOUS` | Flag — confirm exclusion |
| `anvil.yaml` present, name does **not** match convention | `CODE_REPO_AMBIGUOUS` | Flag — confirm exclusion |
| Neither condition met | Not a code repo | Stays in scope |

#### 1.2.4 Name-Pattern Exclusions (anywhere in scope, case-insensitive substring match)

Any folder whose name contains any of the following is excluded, along with its contents:

- `obsolete`
- `backup` (or any variation/spacing thereof)
- `archive`
- `gbrain`, `gstack`, `opencode` (or any variation/spacing thereof)

#### 1.2.5 Exceptions to 1.2.4

The following are force-included even though they match the `gbrain` / `gstack` / `opencode`
pattern exclusion — these are reference documentation, not the tools themselves:

```
C:\projects-reference\workspace-reference\gstack-reference
C:\projects-reference\workspace-reference\opencode reference
C:\projects-reference\workspace-reference\gbrain-reference
```

### 1.3 Pinned Documents (bespoke update rules — Section 3.1)

These are singular, canonical, one-of-a-kind files. They are identified by fixed path because
their update rules depend on structural content that doesn't generalize to "any file with this
name." If not found at the stated path, that itself is a `BROKEN`/flagged condition — it is not
silently treated as "moved" or "doesn't exist yet."

| Document | Type | What it tracks |
|---|---|---|
| `C:\dev\dev-root\docmap.md` | Markdown tree | Full division folder hierarchy, file placement rules — the **root** docmap |
| `C:\dev\dev-root\project-inventory.md` | Markdown registry | Project paths, GitHub repos and remotes, GBrain sources, GStack artifacts |

**Retired:** `scaffold-system.html` was a one-time HTML visual companion to `docmap.md`, used
during an early reorganization. It is no longer a managed document — once its badges settle to
"existing," it becomes a lower-density restatement of what `docmap.md` already states directly,
and maintaining two parallel representations of the same territory is unnecessary duplication.
The last version has been archived to `C:\mybizz\archive\` as a historical record of that
reorganization; Docs Manager no longer expects it to exist, verifies it, or reports it missing.

### 1.4 Discovered Documents (filename-based, anywhere in scope)

No hardcoded list. Any file with one of these names, found anywhere under the in-scope walk
(Sections 1.1–1.2), is included:

| Filename | Ruleset applied |
|---|---|
| `README.md` | Section 3.5 |
| `AGENTS.md` | Section 3.5 |
| `INDEX.md` | Section 3.3 |
| `docmap.md` **other than** `C:\dev\dev-root\docmap.md` | Section 3.4 (Project Docmap) |

Because discovery is by filename rather than by list, duplicates are expected and handled
normally — e.g. two unrelated `README.md` files existing side by side is not itself an error.
A bare filename **reference** to `README.md` (e.g. a backtick-wrapped mention with no path) that
matches more than one discovered file is classified `AMBIGUOUS` (Section 2.3) and flagged, not
guessed at.

### 1.5 Depth & Read Model

This is the exact answer to "how deep does Docs Manager go, and does it read everything":

| Layer | Depth | What happens |
|---|---|---|
| **Folders** (all of them, in scope) | Unlimited | Existence/structure check only — "does this directory exist, is it represented in the relevant map." Contents are never read just because a folder exists. |
| **The five managed document types** (Section 1.3 + 1.4) | Unlimited | Full read, every instance, wherever it appears in scope. Nothing is summarized or skipped for being numerous — see Section 6, no vague quantifiers. |
| **Every other file** (code, config, data, anything not named above) | N/A | Never opened, never read, at any depth. |
| **References found inside a managed document** | One level | The target's existence is checked. Docs Manager does not chase further into an unrelated target's own content — unless that target is itself one of the six managed types, in which case it's already being scanned as part of the same run anyway. |

---

## 2. Reference Extraction Rules

The sub-agent extracts internal references from every in-scope document (pinned + discovered).
An internal reference is any string that appears to point to another location within the
walked scope.

### 2.1 Path Patterns Recognized

| Pattern | Example | How matched |
|---|---|---|
| Windows absolute | `C:\dev\dev-root\docmap.md` | Regex for `C:\` prefix |
| Windows backslash dir | `C:\mybizz\logs\` | Same, with `\` |
| WSL absolute | `/mnt/c/dev/dev-root/docmap.md` | Regex for `/mnt/c/` prefix |
| WSL home-relative | `~/.config/opencode/AGENTS.md` | Regex for `~/` prefix |
| Slug reference | `` `docmap.md` `` (backtick-wrapped, no path) | Resolved against all discovered/pinned documents by filename |
| Relative from doc | `docs-local/docmap.md` | Resolved relative to document's directory |

### 2.2 What is NOT an Internal Reference

- URLs (`https://...`)
- Anvil paths (`client_code/...`, `server_code/...`)
- Git references (`HEAD`, `main`, commit hashes)
- Tool commands (`gbrain sync --source`) — and note per Hard Constraint 3, Docs Manager itself
  never executes a command like this even in the course of resolving a reference
- Template placeholders (`<slug>`, `<project-library>`, `dev-(SLUG)`)

### 2.3 Resolution Logic

For each extracted reference:
1. Normalize to a canonical Windows path (`C:\...`) using known mount points
2. Check if that path exists in the filesystem
3. If the referent falls within scope, verify the path matches an actual in-scope document
4. If the referent is a directory, verify the directory exists (this applies even to code-repo
   folders — see 2.5 — even though their contents are never opened)
5. Classify as: **VALID**, **BROKEN** (target doesn't exist), **MISMATCHED** (target exists
   but at a different path), or **AMBIGUOUS** (bare filename matches multiple in-scope
   documents)

### 2.4 Circular Reference Exclusion

The sub-agent never scans `C:\mybizz\logs\` or `C:\backup-docs-manager\` (or any subdirectory
of either). Both contain output generated by this skill's own runs — scanning them would cause
circular drift where each run's output mutates the scan surface for the next run.

### 2.5 Content-Excluded vs. Reference-Resolvable

A code repo (Section 1.2.3) is content-excluded — nothing inside it is opened, globbed, or
read. It is **not** reference-invisible: if a pinned or discovered document (most commonly
`docmap.md`) lists the repo's top-level folder path, that path is still existence-checked like
any other reference. This is required so a human or agent reading `docmap.md` can see that the
repo exists and where it lives, even though Docs Manager itself never looks inside it.

**`docmap.md` must list every code repo's top-level folder as an entry.** Verifying this listing
is part of the Division Map / Project Docmap update rules (Section 3.1, 3.4) — a code repo that
exists on disk but is missing from its parent docmap is an `UNDOCUMENTED` finding, same as any
other undocumented folder.

---

## 3. Update Rules per Document Type

Each document type has specific update rules. The sub-agent checks these rules, not a generic
staleness heuristic. **Every `docmap.md` — the pinned root one and every discovered project
one — carries a `verified: <date>` field, proposed for update to today's date on every run,
regardless of whether anything else in it changed.** This goes through the normal Phase 3
approval like any other proposed change (Hard Constraint 1) — it is never written silently.

### 3.1 Pinned Division Documents

**`C:\dev\dev-root\docmap.md`:**
- Walk the filesystem for every directory tree listed (Sections 1–4)
- Verify every directory shown exists
- Verify every directory at those roots is represented (no undocumented directories)
- Verify every code repo (Section 1.2.3) at those roots is listed as an entry (Section 2.5)
- Verify file placement rules (Section 6) cover all current folder types
- Verify tool installations (Section 7) are accurate
- Verify companion documents (Section 8) list current key documents
- Propose updating `verified:` to today's date

**`C:\dev\dev-root\project-inventory.md`:**

This is the single source of truth for repo/remote tracking — it absorbs what would otherwise
be a separate, drift-prone repo inventory. It tracks every git repository in scope in three
categories:

| Category | Definition | What's verified |
|---|---|---|
| **Active Repositories** | Has a `.git` root and a configured remote | Local path exists; remote URL matches (`git remote get-url origin`); branch name matches |
| **Inactive / No Remote** | Has a `.git` root but no remote, or is a known-obsolete repo | Local path exists; status note (e.g. "template, not version-controlled" or "obsolete") is accurate |
| **Installed Tools (read-only)** | Third-party tool repos (GBrain, GStack, Skills) with an upstream remote Docs Manager doesn't own | Local path exists; upstream remote noted for reference only — **never verified against `git status`, never staged, never committed** |

- Verify every project's local path exists
- Check GitHub remote URLs and branch names match (via `git remote get-url origin` /
  `git branch --show-current`)
- Verify GBrain source names match reality (via `gbrain sources list` — read-only, Hard
  Constraint 3)
- Update `updated:` date field if changed

### 3.3 INDEX Files (any `INDEX.md` discovered in scope)

- Verify every file listed in the index exists at the stated path
- Verify no files at the indexed location are missing from the index
- Verify counts (e.g., "16 docs", "40 ADRs") match reality
- Check date stamps

### 3.4 Project Docmaps (any `docmap.md` other than the pinned root docmap)

- Same as division maps but scoped to one project
- Verify all folders and files listed exist
- Verify any code repo within that project's scope is listed (Section 2.5)
- Verify no new files are undocumented within the project scope
- Propose updating `verified:` to today's date

### 3.5 AGENTS.md and README.md (any discovered in scope)

- Scan for internal references and verify they resolve
- Flag any reference to deprecated or relocated paths
- Do NOT evaluate content quality — only structural accuracy

---

## 4. Learnings

### 4.1 Purpose

Docs Manager accumulates institutional memory across runs. When the developer resolves a
judgment call — an `AMBIGUOUS` reference, a `CODE_REPO_AMBIGUOUS` folder, a `MISMATCHED` path
with no obvious fix — that resolution is captured so the same situation isn't re-litigated
from scratch on every future run. This also gives the developer a way to steer the system's
behavior without editing SKILL.md or any agent file directly.

### 4.2 Precedence

**Learnings overrule SKILL.md** for everything except the Hard Constraints at the top of this
document. If a Learnings entry conflicts with a SKILL.md rule on scope, classification,
exclusion, or recommendation logic, the Learnings entry wins. Learnings can never:
- authorize a write outside Phase 4
- authorize skipping Phase 0 backup or its verification
- authorize running `gbrain sync` or any GBrain state-changing command
- bypass Phase 3 developer approval

Within those limits, Learnings are the developer's mechanism for correcting or refining the
system's behavior over time.

### 4.3 Phase −1: Load Learnings

Before anything else runs — before Phase 0 backup even — OpenCode reads every file in
`C:\mybizz\logs\docs-manager\Learnings\` and carries the content forward as context into
Phase 1 (Scan), so recommendations can reference prior precedent ("last time a similar case
came up, you decided X — recommending the same here").

### 4.4 Phase 8: Update Learnings

After Phase 7 (Log), OpenCode reviews the run for judgment calls the developer made in Phase 3
that are likely to recur (anomaly resolutions, new exceptions, corrected assumptions) and
writes one new file per learning to `C:\mybizz\logs\docs-manager\Learnings\`.

### 4.5 Format

One small file per learning, not one growing file:

```
C:\mybizz\logs\docs-manager\Learnings\<timestamp>-<short-topic>.md
```

```markdown
# Learning — <timestamp>

**Context:** <what situation triggered this>
**Decision:** <what the developer decided>
**Applies to:** <what future situations this should inform>
```

---

## 5. Execution Workflow

**Invocation is the trigger, not a request to begin one.** When the developer runs
`/docs-manager` (or `/docs-manager <scope>`), Phase −1 and Phase 0 start immediately — no
"would you like me to begin?", no "should I proceed?", no confirmation step of any kind. The
orchestrator does not address the developer again until one of exactly two things happens: (a)
Phase 0 aborts (`REPOS_DIRTY` or `BACKUP_FAILED`) and reports why, or (b) Phase 2 presents the
change report for approval. Asking permission to start, or asking which scope was meant when a
valid scope keyword (or no keyword, meaning full run — Section 7) was already given, is treated
as a failure to follow this section, not a helpful clarification. If `/docs-manager` is invoked
with no scope keyword, that means full run (Section 7) — this is not ambiguous and must not be
asked about.

### Phase −1: Load Learnings — Orchestrator (read-only)

See Section 4.3. Not a sub-agent — OpenCode reads the Learnings folder directly and passes it
forward. Begins immediately on invocation, per the rule above.

### Phase 0: Pre-flight Check & Backup — Sub-agent (write, copy only)

Runs automatically — no per-run approval needed, per developer agreement (it only ever creates
new copies, never touches an original).

#### Step 0: Pre-flight git status check

Before anything else, check `git status` on every **Active Repository** listed in
`project-inventory.md` (Section 3.1) — this deliberately excludes code repos (Section 1.2.3,
content-excluded and never committed to by Docs Manager anyway) and Installed Tools (never
touched, Section 3.1).

If any in-scope repository has uncommitted changes, **abort the entire run before backup or
scan begins**. Report: *"The following repositories have uncommitted changes: [list]. Docs
Manager cannot run while repos are out of sync — please commit or clean git status first."*
This is the `REPOS_DIRTY` anomaly (Section 9) and is not recoverable automatically — the
developer must resolve it outside Docs Manager and re-invoke the run.

This precondition exists so Phase 6 never has to decide what to do with unrelated pending
changes sitting in a repo — if the run starts, every in-scope repo is already known to be
clean, and only Docs Manager's own changes will ever appear in a commit.

#### Step 1: Backup

1. Compute the full in-scope file set (Section 1.1 minus Section 1.2) — the same set Phase 1
   will walk.
2. Copy that entire set to `C:\backup-docs-manager\<run-timestamp>\`, preserving relative
   paths.
3. Verify the copy: compare file count and total size (or checksums, if available) between
   source scope and backup.
4. If verification fails for any reason, **abort the entire run** (Hard Constraint 2). No
   scan, no report, no changes. Return a failure notice to the developer with the
   `BACKUP_FAILED` reason.
5. Only on verified success does the run proceed to Phase 1.

`C:\backup-docs-manager\` itself is excluded from all future scan scope (Section 2.4) so
backups never get walked, referenced, or treated as drift.

### Phase 1: Scan — Sub-agent (read-only)

Runs only after Phase 0 has verified successfully. OpenCode launches a sub-agent, providing it
the Learnings content from Phase −1. The sub-agent:

1. Reads every pinned document (Section 1.3)
2. Walks the in-scope filesystem (Section 1.1 minus 1.2) and discovers every README.md,
   AGENTS.md, INDEX.md, and non-root docmap.md (Section 1.4)
3. Reads every discovered document
4. Extracts all internal references from every document (pinned + discovered)
5. Resolves every reference against the filesystem, including code-repo folder references
   (Section 2.5, existence-check only)
6. Applies document-type-specific update rules (Section 3), including proposing `verified:`
   date updates
7. Counts folders walked and files inspected (Section 6 — required in the report and log)
8. Returns a change report, with a concrete recommendation attached to every flagged item

### Phase 2: Report — Presented to Developer

```markdown
# docs-manager Change Report — <timestamp>

## Summary
- Folders walked: N
- Files inspected (managed types): N
- Documents checked: N
- References scanned: N
- Changes proposed: N
- Anomalies flagged: N

## Proposed Changes

### <Document Path>
| Line | Current | Proposed | Reason |
|---|---|---|---|
| ... | ... | ... | ... |

## Flagged Anomalies (Needs Human Decision)

### <Document Path>
| Location | Issue | Recommendation |
|---|---|---|
| ... | ... | ... |

## Newly Discovered Documents
| Document | Location |
|---|---|
| ... | ... |
```

Every row in "Flagged Anomalies" must carry a concrete recommendation — "here's the issue, here
is what I'd do" — never just a question with no suggested answer (Section 6).

### Phase 3: Approval — Developer Reviews

OpenCode presents the change report. Developer reviews proposed changes and anomalies. Approval
is per-document, per-change, or batch. **Approval triggers continuation** — saying "approved"
or "apply" moves to Phase 4.

Resolved anomalies are treated as approved changes for Phase 4, and are candidates for a new
Learnings entry (Phase 8).

### Phase 4: Apply — Sub-agent (write)

A second sub-agent applies only the approved changes (Hard Constraint 1). Unresolved anomalies
are not applied.

If no changes were approved: skip to Phase 7 (Log).

### Phase 5: Verify — Sub-agent (read-only)

If changes were applied, the sub-agent:
1. Re-reads every changed document
2. Re-extracts and resolves all internal references
3. Confirms no new broken references were introduced
4. Returns a one-line verification result

### Phase 6: Commit & Push — Sub-agent (write, git)

Runs only if Phase 4 applied changes and Phase 5 verified cleanly. If no changes were applied
in Phase 4, skip this phase entirely.

Because Phase 0's pre-flight check (Step 0) already guaranteed every in-scope repo was clean
before the run started, the only changes present in any repo at this point are Docs Manager's
own — there is no unrelated pending content to reason about.

#### Step 1: Discover and resolve

Group changed files by the git repository they belong to (walk upward from each changed file
to the nearest `.git` root).

- If no `.git` directory is found upward from a changed file: classify as `NO_REPO`, skip git
  operations for it, and report it under "Skipped — Not a Git Repository." This is expected
  and correct for locations like `project-template/` (template, not version-controlled) — it
  is reported for visibility, not treated as an error.
- If a `.git` directory is found: resolve the exact git-tracked filename for each changed file
  using `git ls-files --full-name <filename>` inside that repository, rather than the on-disk
  filename. This handles NTFS case-insensitivity, where the on-disk filename can differ in
  casing from the git-indexed filename (e.g., `agents.md` on disk vs `AGENTS.md` in git) —
  using the on-disk name with `git add` can silently miss the file.

#### Step 2: Stage and commit (scoped — Docs Manager changes only)

`git add` only the resolved, Docs-Manager-changed files in each repository — never `git add -A`
or any broad stage. Docs Manager's commits contain exactly what it changed, nothing else, per
Hard Constraint 1. (The Phase 0 pre-flight check is what makes this safe to do narrowly — there
should be nothing else pending in the repo to leave behind.)

Commit with message: `"docs-manager: <run-timestamp> — N changes applied"`.

This commit happens regardless of remote status (see Step 3) — a local commit is real,
recoverable history even if the remote is unreachable, and should not be skipped just because
the push might fail.

#### Step 3: Validate remote and push

For each repository committed in Step 2: run `git ls-remote --heads origin` to confirm the
remote is reachable before pushing.

- If this fails (no remote configured, repository not found, authentication failure): do not
  retry silently. Report `PUSH_FAILED` for that repository with the exact error. The local
  commit from Step 2 remains in place regardless — it is not discarded.
- If it succeeds: run `git push`. If the push itself then fails (conflict, network error,
  etc.), same handling — report `PUSH_FAILED`, keep the local commit.
- The orchestrator surfaces any `PUSH_FAILED` result to the developer immediately, not just in
  the eventual log — remote issues (missing GitHub repo, wrong remote URL) often need the
  developer's attention before the run can be considered closed.

#### Step 4: Write the commit/push report

Return a per-repository result table to the orchestrator:

| Repository | Commit | Files Changed | Push Result |
|---|---|---|---|
| ... | ... | ... | PUSHED / PUSH_FAILED / NO_REPO |

The orchestrator writes this to `C:\mybizz\logs\github-logs\commit-push-<date>.md` — a separate
directory from `C:\mybizz\logs\docs-manager\`, so git-focused logs never get mixed into
docs-manager run logs. The Phase 7 log's own "Commit/Push Detail" table should reference this
file rather than duplicating it in full, so there's one source of truth for the detail.

### Phase 7: Log — Orchestrator

OpenCode writes the log entry to `C:\mybizz\logs\docs-manager\<timestamp>.md`.

**The `<timestamp>` is when the log is written — run completion time — not when Phase 1
started.** Generate it fresh at write time from the current system clock. A run spanning two
calendar days (scan on day 1, approval and apply on day 2) is named for the day it concluded,
not the day it began. The log body records both times as internal fields:

```markdown
# docs-manager run — 2026-07-23T10-47-00

**Scope:** full
**Scan started:** 2026-07-22T17-27-00
**Completed:** 2026-07-23T10-47-00
**Backup:** verified — C:\backup-docs-manager\2026-07-23T10-47-00\
**Folders walked:** 340
**Files inspected (managed types):** 27
**Documents checked:** 27
**References scanned:** 61
**Changes proposed:** 5
**Changes approved:** 5
**Changes applied:** 5
**Anomalies flagged:** 2
**Verification:** all references resolve
**Commit/Push:** see C:\mybizz\logs\github-logs\commit-push-2026-07-23.md

## Applied

| Document | Change | Reason |
|---|---|---|
| ... | ... | ... |

## Flagged (Not Applied)

| Document | Issue | Recommendation |
|---|---|---|
| ... | ... | ... |
```

No narrative, no planning — structured event record only. No vague quantifiers (Section 6) —
folder and file counts are exact, every time.

### Phase 8: Update Learnings — Orchestrator (mandatory)

**Phase 8 is mandatory whenever Phase 4 applied changes. The orchestrator must not skip it.**

After the log is written (Phase 7), the orchestrator reviews the run for judgment calls the
developer made in Phase 3 that are likely to recur — anomaly resolutions, new conventions
established, edge cases the skill text doesn't yet address — and writes one file per learning
to `C:\mybizz\logs\docs-manager\Learnings\<timestamp>-<short-topic>.md` (Section 4.5 format),
using the same `<timestamp>` as the Phase 7 log.

**Mandatory verification:** after writing learnings, the orchestrator must confirm at least one
new file exists in the Learnings folder matching the current run's timestamp
(`ls C:\mybizz\logs\docs-manager\Learnings\<timestamp>-*.md`). If none is found, Phase 8 was
skipped — this is a process failure, not a silent no-op. Report the omission to the developer
and offer to create learnings now before considering the run closed.

If the run applied changes but genuinely produced no novel judgment calls (everything followed
existing SKILL.md rules and prior Learnings entries), write a single minimal learning stating
exactly that, so the verification step above still finds a file and the record shows Phase 8
executed:

```markdown
# Learning — <timestamp>

**Context:** All decisions in this run were standard per existing SKILL.md rules and
prior Learnings entries. No new judgment calls occurred.

**Decision:** No new learnings to record.

**Applies to:** Future runs — demonstrates that Phase 8 executed.
```

### Phase 9: Completion Gate — Orchestrator

After Phase 8 completes and is verified, before the run is considered closed:

#### Step 1: Re-verify the division map

Run a small, `division-maps`-scoped re-scan of `docmap.md` and `project-inventory.md` only
(Section 7). This exists because the run's own actions (Phase 4 edits, Phase 6 commits) can
themselves cause these two documents to drift by the time the run finishes — e.g. a new
directory or repo appearing as a side effect of what was just applied. If this re-scan finds
anything, it goes through its own small Phase 2/3/4 report-approve-apply cycle before
continuing — it does **not** write silently, same as every other change in this system.

#### Step 2: GitHub confirmation prompt

Ask the developer, as its own separate question (never combined with Step 3):

- If Phase 6 already ran this session: *"Commit and push to GitHub completed. All repositories
  showing clean. Confirm?"*
- If Phase 6 was skipped (e.g. `/docs-manager no-push` scope was used): *"Phase 6 was skipped.
  Would you like to commit and push to GitHub now?"*

#### Step 3: GBrain sync prompt

Ask, as its own separate question: *"Would you like to run the GBrain sync script?"*

This is an optional, explicitly opt-in courtesy offered by the orchestrator after Docs Manager
has finished — it is not something Docs Manager does as part of its own workflow, and it must
never be assumed or run automatically (Hard Constraint 3). If the developer says yes, the
orchestrator — not any Docs Manager sub-agent — launches the external sync script:

```
setsid bash /mnt/c/mybizz/scripts/sync-gbrain.sh > /tmp/sync-output.txt 2>&1 &
```

The script handles the stop/sync/embed/restart cycle externally and writes its own log to
`C:\mybizz\logs\gbrain-logs\sync-<timestamp>.log`. Docs Manager itself never executes `gbrain
sync` or touches GBrain state directly.

#### Step 4: Closing git-status snapshot

After both prompts are answered, run a `git status` sweep across all Active Repositories
(Section 3.1) and save the result to `C:\mybizz\logs\github-logs\git-status-<date>.md` (date
only, not a timestamp — distinguishes it from the commit-push report). The file contains a
table: repository, remote URL, branch, ahead/behind counts, and status (clean or dirty). This
is the definitive closing record that every repo is confirmed synced — a fixed point of
reference if a discrepancy is ever suspected later.

---

## 6. Reporting Standard (applies to every phase's output)

- **No vague quantifiers.** Never "many folders," "several files," "and others." Every folder
  and every managed document is named individually. If a table is genuinely too long for one
  block, split it into multiple clearly-labeled tables — never collapse into vague language.
  The managed scope is small enough that this is never actually a volume problem in practice.
- **Every flagged item carries a recommendation**, not just a description of the problem.
- **Exact counts, every time** — folders walked and files inspected are always stated as
  numbers, in both the Phase 2 report and the Phase 7 log.

---

## 7. Scope Parameter

`/docs-manager` alone — no keyword after it — **means full run, unconditionally.** This is a
default, not a gap to ask about (Section 5).

- `/docs-manager` — full run (backup → scan all pinned + discovered documents → ... → commit/push → log → learnings)
- `/docs-manager <document-path>` — scan only that one document (backup still runs for the full
  in-scope set, since drift elsewhere may still affect that document's references)
- `/docs-manager division-maps` — scan only Section 1.3 (pinned documents)
- `/docs-manager quick` — scan only Section 1.3 plus discovered README.md files (high-value,
  most frequently drifted)
- `/docs-manager references` — scan all documents for internal reference resolution only (no
  directory walk)
- `/docs-manager no-push` — run through Phase 5 (Verify) and Phase 7 (Log), skipping Phase 6
  (Commit & Push) — useful for local-only review runs

Whichever of these was invoked, execution begins immediately per Section 5 — no confirmation,
no "which of these did you mean."

---

## 8. Sub-Agent Files

Four single-purpose agent files, each with only the permissions its phase needs:

| File | Phase | Permissions |
|---|---|---|
| `docs-manager-backup.md` | 0 — Backup | copy-only |
| `docs-manager-scan.md` | 1 — Scan | read-only |
| `docs-manager-apply.md` | 4 — Apply approved changes | edit |
| `docs-manager-commit.md` | 6 — Commit & Push | git/bash |

Phases −1, 2, 3, 7, 8, and 9 are orchestrator-level (OpenCode itself), not sub-agents — they're
simple reads/writes or developer interaction, not scanning work. (Phase 9's Step 1 re-verify
does launch the Phase 1 scan sub-agent again, scoped to `division-maps` — it isn't a new
sub-agent, just a repeat, narrower invocation of the existing one.)

**None of these files pin a specific model.** The frontmatter intentionally omits a `model:`
field so each run uses whichever model is currently active/selected — a model named explicitly
today is likely to be outdated later, and this system should not need editing just because the
active model changes.

Prompt content for each is in `docs-manager-backup.md`, `docs-manager-scan.md`,
`docs-manager-apply.md`, and `docs-manager-commit.md` respectively — see those files for the
full instructions each sub-agent runs under. They implement Sections 1–6 of this document.

---

## 9. Anomaly Classification

| Class | Definition | Action |
|---|---|---|
| **REPOS_DIRTY** | Phase 0 pre-flight found uncommitted changes in an in-scope Active Repository | Abort entire run before backup or scan — developer must clean git status and re-invoke |
| **BACKUP_FAILED** | Phase 0 backup could not be verified against source scope | Abort entire run — no scan performed |
| **BROKEN** | Reference points to a path that doesn't exist and no similar path exists | Propose fix if target is known, else flag with recommendation |
| **MISMATCHED** | Reference points to a path, but the known correct path for that document is different | Propose correct path |
| **AMBIGUOUS** | Bare filename matches multiple discovered documents | Flag — recommend the most likely match, ask developer to confirm |
| **CODE_REPO_AMBIGUOUS** | Folder matches only one of the two code-repo signals (naming convention, `anvil.yaml`) | Flag — recommend a resolution, ask developer to confirm |
| **STALE DATE** | Document has a date field older than 7 days since last run | Flag — not actionable automatically |
| **MISSING** | Directory or file expected by the document doesn't exist on disk | Propose removal or flag with recommendation if uncertain |
| **UNDOCUMENTED** | Directory or file (including a code repo — Section 2.5) exists on disk but not in the document | Propose addition |
| **COUNT MISMATCH** | INDEX or doc states a count (e.g., "16 docs") that doesn't match reality | Propose updated count |
| **NO_REPO** | A changed file has no `.git` root found upward from it | Report under "Skipped — Not a Git Repository" — expected for templates, not an error |
| **PUSH_FAILED** | Phase 6 commit succeeded locally but `git push` failed (or remote validation failed before push) | Flag immediately to developer — commit stands, push must be retried manually |

---

## 10. Log Directory Structure

Three directories under `C:\mybizz\logs\`, three naming conventions, no cross-contamination:

```
C:\mybizz\logs\
├── docs-manager\
│   ├── <timestamp>.md                    ← Phase 7 run log (bare timestamp, completion time)
│   └── Learnings\
│       └── <timestamp>-<short-topic>.md  ← Phase 8 learnings
├── github-logs\
│   ├── commit-push-<date>.md             ← Phase 6 commit/push report
│   └── git-status-<date>.md              ← Phase 9 closing snapshot
└── gbrain-logs\
    └── sync-<timestamp>.log              ← Written by the sync script itself, not Docs Manager
```

`github-logs/` is a new directory not yet reflected in `docmap.md` at the time of this
revision — the first run after this change will find it `UNDOCUMENTED` under the normal
Section 3.1 rules and propose adding it, same as any other new folder.

---

## 11. Edge Cases

1. **Circular references**: If doc A references doc B and doc B references doc A, both are
   valid if both resolve. No special handling.

2. **WSL/Windows duality**: All paths are normalized to `C:\...` for comparison. WSL paths
   in documents are flagged as "consider standardizing" but not broken unless the target
   doesn't exist.

3. **Self-references**: A document referencing itself — ignored.

4. **Section references**: `docmap.md Section 6` — file-level resolution only. Section number
   anchoring is a possible future feature.

5. **Template files**: Documents in `project-template\`, `obsolete\dev-project-template\` are
   excluded by the `obsolete` name-pattern rule (Section 1.2.4) and are not scanned. If a
   non-obsolete template folder is added later, it is scanned for references, but directory
   walk expectations are relaxed — templates contain placeholders by design.

6. **One run at a time**: Sequential execution. Phase 0 completes and verifies before Phase 1
   starts; the scan builds the full document set first, then resolves references — no parallel
   checks because each document's references may depend on the full set.

7. **No auto-persist of judgment calls**: Newly discovered documents are reported every run —
   this is expected, not flagged as drift. `CODE_REPO_AMBIGUOUS` folders are never auto-resolved
   in either direction; the developer decides each time until the folder is fixed (renamed or
   given/denied `anvil.yaml`), though a Learnings entry can make the recommendation consistent
   across runs.

8. **The log and learnings files themselves**: Written by OpenCode (Phases 7 and 8), not by any
   sub-agent. No sub-agent ever writes to `C:\mybizz\logs\`.

9. **Backup growth**: `C:\backup-docs-manager\` accumulates one timestamped snapshot per run
   and is never itself scanned or pruned by this skill. Retention/cleanup of old backups is a
   separate, manual concern — confirmed as an accepted tradeoff.

10. **Repo-internal docs stay opaque, but the repo's existence doesn't**: Once a folder is
    classified as a code repo (Section 1.2.3), nothing inside it — including its own
    README.md — is discovered, read, or reference-checked. Its top-level path must still
    appear in the relevant docmap.md (Section 2.5), and that listing is verified. Its sibling
    `project-library` (or equivalently named) folder remains fully in scope as normal.

11. **Partial commit/push failure**: If changes span multiple repositories and one repo's push
    fails, the others still commit and push independently — Phase 6 does not roll back on a
    per-repo failure elsewhere.

12. **Learnings vs. SKILL.md conflicts**: Handled per Section 4.2 — Learnings win on
    substantive/judgment matters, Hard Constraints always win regardless of source.

13. **`scaffold-system.html` retirement**: No longer a pinned document (Section 1.3). If a file
    still exists at its old path, Docs Manager does not read, verify, or report on it — it is
    simply out of scope going forward, same as anything else not named in Section 1.3/1.4. Its
    last version lives in `C:\mybizz\archive\` as a historical record only.

14. **Repo inventory is not a separate document**: `project-inventory.md` is the single source
    of truth for git repos, remotes, and branches (Section 3.1). If a separate repo-inventory
    file is ever found elsewhere in scope, it is flagged as `UNDOCUMENTED`/redundant rather than
    read as an independent source — two documents tracking the same repo list is exactly the
    kind of duplication this system exists to eliminate.

15. **A dirty repo blocks the entire run, not just its own commit**: Unlike a single anomaly,
    `REPOS_DIRTY` (Section 9) is a precondition failure, not a scan finding — it stops the run
    before Phase 0's backup even begins. This is deliberate: it's simpler and safer to guarantee
    every in-scope repo starts clean than to have Phase 6 reason about what to do with unrelated
    pending changes mid-run.
